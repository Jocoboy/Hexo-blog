---
title: 领域驱动设计与ABP框架
date: 2024-08-13 12:54:29
categories:
- Framework
tags:
- DDD
- ABP
---

领域驱动设计核心概念与ABP框架实践。

<!--more-->

## 前言

ABP框架是对领域驱动设计DDD的一种实现，主要目标是为应用程序开发引入的一种架构方法。

## 领域驱动设计

领域驱动设计(Domain Driven Design)是一种针对复杂需求的软件开发方法，它适用于复杂领域和大规模应用，关注核心领域逻辑而不是基础设施细节。

### 基本分层

DDD分层设计包含表现层(Presentation Layer)、应用层(Application Layer)、领域层(Domain Layer)、基础设施层(Infrastructure Layer)四个部分。

- 表现层包含应用的UI组件，通过API网关连接前端应用和后端微服务
- 应用层用于协调表现层和领域层，不包含领域逻辑，只负责调用领域层的功能
- 领域层包含基本的业务对象，是独立的可重用的领域逻辑，不依赖于任何层
- 基础设施层提供数据持久化、消息传递和第三方集成等服务，包括数据库访问层、消息队列等

领域层和应用层是业务逻辑的核心部分。

#### 应用层

应用层包含应用服务、数据传输对象(DTO)、工作单元(UOW)等基本概念。

- 数据传输对象用于表现层和应用层之间传输数据
- 工作单元是事务边界，其中的所有状态更改必须以原子方式实现

#### 领域层

领域层包含实体(Entity)、值对象(Value Object)、聚合(Aggregate)和聚合根(AggregateRoot)、仓储(IRepository)、领域服务(Domain Service)、领域事件(Domain Event)等基本概念。

- 值对象与实体不同之处在于，没有唯一标识符，如果两个值对象的所有属性都相同，则它们被认为是相同的(c# 9.0中record新特性与此类似)
- 聚合根通常是一个具有唯一标识(‌如GUID)的实体，‌它负责协调聚合内部的其他实体和值对象，‌确保数据的一致性和完整性
- 领域服务是实现核心业务规则的无状态服务(类命名通常以Manager为结尾)，它的实现依赖于多种聚合或外部服务


## ABP框架

在使用ABP框架时，需要特别注意项目之间的引用和依赖关系。

### 设计演变

按照DDD的最初设计，ABP框架应当包含以下四个部分

- [company].[application].Web
- [company].[application].Application
- [company].[application].Domain
- [company].[application].Infrastructure

基础设施层通常需要包含一种对象关系映射(ORM)方式，ABP中使用了EntityFrameworkCore，
由此框架演变为

- [company].[application].Web
- [company].[application].Application
- [company].[application].Domain
- [company].[application].EntityFrameworkCore

由于Web层直接引用Application，而Application直接引用Domain，这将导致Application间接引用Domain，这不符合抽象和实现分层原则，因此ABP引入了Application.Contracts

- [company].[application].Web
- [company].[application].Application.Contracts
- [company].[application].Application
- [company].[application].Domain
- [company].[application].EntityFrameworkCore

由于我们将接口定义和DTO存放在了Application.Contracts，而DTO有时需要重用Domain中的枚举类型，但Application.Contracts又无法直接添加对Domain的引用，为了解决这个问题，ABP引入了Domain.Shared

- [company].[application].Web
- [company].[application].Application.Contracts
- [company].[application].Application
- [company].[application].Domain
- [company].[application].Domain.Shared
- [company].[application].EntityFrameworkCore

为了将REST API与UI分离，ABP引入了HttpApi，此外还引入了HttpApi.Client客户端代理系统和数据库迁移工具DbMigrator，由此完整的项目框架为

- [company].[application].Web
- [company].[application].HttpApi
- [company].[application].HttpApi.Client
- [company].[application].Application.Contracts
- [company].[application].Application
- [company].[application].Domain
- [company].[application].Domain.Shared
- [company].[application].EntityFrameworkCore
- [company].[application].DbMigrator

{% asset_img abp_layer_deps.png ABP框架中各层之间的依赖关系 %}

从上图可以看出，Domain层无法直接使用Application.Contracts中的DTO，必要时可使用Domain.Shared中的Value Object代替DTO。

此外，ABP框架中test文件夹中还包含了每一层单独配置的单元/集成测试项目。

### 实现细节

[演示项目](https://github.com/Jocoboy/Jocoboy.ABPDemo)

#### 数据库迁移

EFCore提供了一套原生的数据库迁移系统，基于Code First原则。在Domain层创建好相关实体后，打开Nuget Package Manager Console输入以下命令即可自动完成数据库迁移

`Add-Migration InitialCreate`

`Update-Database`


## 参考文档

- [C# 9.0 新特性](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history#c-version-9)
- [ABP官方文档](https://abp.io/docs/latest/)
- [EFCore数据库迁移](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/?tabs=vs)
