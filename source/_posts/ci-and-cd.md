---
title: 持续集成、交付与部署CI/CD及Jenkins
date: 2024-08-25 20:41:57
categories:
- CI/CD
tags:
- Devops
- Jenkins
- Docker
---

持续集成、交付与部署CI/CD基本概念、流程，以及使用Jenkins实现自动化部署。

<!--more-->

## 前言

CI/CD是持续集成(Continuous Integration，CI)、持续交付(Continuous Delivery，CD)与持续部署(Continuous Deployment，CD)的简称。

CI/CD是实现敏捷开发和Devops理念的一种方法，可让持续自动化和持续监控贯穿于应用的整个生命周期。这些关联的事务通常被统称为CI/CD管道(Pipeline)，由开发(RD)、测试(QA)、运维(OP)团队以敏捷方式协同支持。

## 基本概念

### 持续集成

{% asset_img ci_step.png 持续集成流程图 %}

持续集成(CI)指的是高频率地将代码合入主干，在合入之前触发单元测试去验证代码的改动，确保改动不会对应用造成破坏。

持续集成强调开发人员提交了新的代码后，立刻进行构建、(单元)测试。根据测试结果，我们可以确定新代码能否和原代码正确地集成在一起。

### 持续交付

{% asset_img cd_step.png 持续交付流程图 %}

持续交付(CD)指的是频繁地将软件的新版本，交付给质量团队或者用户、以供评审。如果评审通过，代码就会进入生产阶段。持续交付的目标是拥有一个可随时部署到生产环境的代码库。

持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的类生产环境中。比如完成单元测试后，可以将代码部署到连接数据库的Staging环境中进行更多的(集成)测试。如果代码没有问题，可以继续手动部署到生产环境中。


### 持续部署

{% asset_img cd_step_2.png 持续部署流程图 %}

持续部署(CD)是持续交付的下一步，指的是代码通过评审后，自动部署到生产环境。由于在生产之前的管道阶段没有手动门控，因此持续部署在很大程度上都得依赖精心设计的自动化测试。


## CI/CD流程

根据CI/CD的设计，代码从提交到生产，有以下几个步骤：

- 提交：开发者向代码仓库提交一次代码
- 测试(第一轮)：代码仓库对commit操作配置了hook，只要提交了代码或者合并到主分支，就会跑自动化测试。测试的种类分为：
  - 单元测试(针对函数或模块的测试)
  - 集成测试(针对产品的某个功能的测试)
  - 端到端测试(从用户界面直达数据库的全链路测试)
- 构建：将源码转换为可以运行的实际代码，如安装依赖、配置各种资源等。常用的构建工具有：
  - Jenkins
  - Travis CI
  - GitLab CI/CD
- 测试(第二轮)：对第一轮测试的补充，可省略
- 部署：将可部署的版本中的所有文件打包到生产服务器上，生产服务器将打包文件解包成本地目录，再将运行路径符号链接指向这个目录，然后重新启动应用。
- 回滚：一但当前版本发生问题，就要回滚到上一个版本的构建结果。最简单的做法是修改链接符号，指向上一个版本的目录


## Jenkins

Jenkins是一个开源的实现持续集成的工具。Jenkins能实时监控集成中的存在的错误，提供详细的日志文件和提醒功能，还能用图表的形式展示项目构建的趋势和稳定性。

### 安装与配置

使用Docker下载Jenkins镜像

`docker pull jenkins/jekins:lts`

创建目录并更改权限

`mkdir -p /mydata/jenkins_home`

`chmod 777 /mydata/jenkins_home`

运行Jenkins容器

`docker run -di --name=jenkins -p [host-port]:[container-port] -v /mydata/jenkins_home/:/var/jenkins_home/ jenkins/jekins:lts`

查看Jenkins运行是否成功

`docker ps -a`

### 创建管理员用户

通过docker启动日志获取Jenkins控制台解锁密码

`docker logs jenkins`

输入解锁密码后填写信息创建一个管理员用户。

### 安装插件

可通过Jenkins控制台的系统管理>插件管理在线安装插件，也可到官网下载hpi插件文件然后上传使用。

### 全局工具配置

可通过Jenkins控制台的系统管理>全局工具配置，配置JDK、Git、Maven等运行环境。

### 配置SSH

- 通过Jenkins控制台安装SSH插件
- 通过凭据>全局添加服务器账号密码的凭据
- 通过系统管理>系统配置，添加SSH remote hosts，主机名为服务器IP，端口默认22

### 创建自动构建任务

可通过Jenkins控制台的新建任务>构建一个自由风格的软件项目新建一个任务，在源码管理添加Git远程仓库(需要添加Git账号密码凭据)，构建环境选择Delete workspace before build starts，并增加构建步骤(添加执行shell，或添加使用SSH在远程服务器上执行脚本)。

```sh
#!/bin/sh
app_name=[jenkinsdemo]
cd /var/lib/jenkins/workspace/${app_name}/[project_name]
docker container prune << EOF
y
EOF
docker container ls -a | grep "${app_name}"
if [ $? -eq 0 ];then
    docker container stop ${app_name}
    docker container rm ${app_name}
fi
docker image prune << EOF
y
EOF
docker build -t ${app_name} .
docker run -d --name=${app_name} -p [host-port]:[container-port]  ${app_name}
```

## 参考文档

- [Jenkins官方文档](https://www.jenkins.io/zh/doc/)