---
title: Unity脚本生命周期
date: 2020-01-04 23:07:14
categories:
- Unity
- C#
tags:
- Unity-Scripting
---

```
Running a Unity script executes a number of event functions in a predetermined order.
```
一个Unity脚本中的事件函数将会以预定顺序执行。
<!--more-->

##  脚本生命周期

### 流程图

下图总结了脚本生命周期中事件函数的排序和重复出现情况。

<img src="Unity-Script-Lifecycle/monobehaviour_flowchart.svg" alt="monobehaviour flowchart">

### 说明

- **Awake**:
    ```
    This function is always called before any Start functions and also just after a prefab is instantiated.
    ```
    此函数将在预制件实例化之后调用，并始终优先于任何Start函数。

- **OnEnable**:
    ```
    This function is called just after the object is enabled. This happens when a MonoBehaviour 
    instance is created, such as when a level is loaded or a GameObject with the script component 
    is instantiated.
    ```
    此函数将在启用对象后立即调用。在创建MonoBehaviour实例时（例如加载关卡或一个携带脚本组件的游戏对象被实例化时）会执行此调用。

- **Reset**:
    ```
    Reset is called to initialize the script’s properties when it is first attached to the object
    and also when the Reset command is used.
    ```
    此函数将在脚本首次被附加到对象上以及使用`Reset`命令时调用，以初始化脚本的属性。

- **Start**:
    ```
    Start is called before the first frame update only if the script instance is enabled.
    ```
    此函数将紧跟Awake函数调用，确切来说是在第一此帧更新前并且此时脚本实例已被启用。

- **FixedUpdate**:
    ```
    FixedUpdate is often called more frequently than Update. 
    It can be called multiple times per frame, if the frame rate is low and it may not be called 
    between frames at all if the frame rate is high. 
    All physics calculations and updates occur immediately after FixedUpdate. 
    When applying movement calculations inside FixedUpdate, you do not need to multiply your values 
    by Time.deltaTime. 
    This is because FixedUpdate is called on a reliable timer, independent of the frame rate.
    ```
    调用FixedUpdate的频度常常超过Update。如果帧率很低，可以每帧调用该函数多次；如果帧率很高，可能在帧之间完全不调用该函数。在FixedUpdate之后将立即进行所有物理计算和更新。在FixedUpdate内应用运动计算时，无需将值乘以 `Time.deltaTime`。这是因为FixedUpdate的调用基于可靠的计时器（独立于帧率）。

- **Update**:
    ```
    Update is called once per frame. It is the main workhorse function for frame updates.
    ```
    此函数是帧更新的主函数，受当前渲染物体和机器性能影响，被调用的时间间隔不固定。

- **Coroutines**：

    ```
    Normal coroutine updates are run after the Update function returns. A coroutine is a function 
    that can suspend its execution (yield) until the given YieldInstruction finishes. 
    Different uses of Coroutines:

    - yield The coroutine will continue after all Update functions have been called on the next frame.
    - yield WaitForSeconds Continue after a specified time delay, after all Update functions have 
    been called for the frame
    - yield WaitForFixedUpdate Continue after all FixedUpdate has been called on all scripts
    - yield WWW Continue after a WWW download has completed.
    - yield StartCoroutine Chains the coroutine, and will wait for the MyFunc coroutine to complete first.
    ```
    Update函数返回后将运行正常协程更新。协程是一个可暂停执行(yield)直到给定的YieldInstruction达到完成状态的函数。协程的不同用法：
    - **yield** 协程将在下一帧上调用所有Update函数后，继续执行。
    - **yield WaitForSeconds** 协程将在指定的时间延迟后（并且当前帧所有Update函数执行后），继续执行。
    - **yield WaitForFixedUpdate** 协程将在所有脚本的FixedUpdate函数执行后，继续执行。
    - **yield WWW** 协程将在WWW资源下载完成后，继续执行。
    - **yield StartCoroutine** 协程将在指定协程执行后，继续执行。

- **LateUpdate**:
    ```
    LateUpdate is called once per frame, after Update has finished. 
    ```
    此函数将紧跟Update函数调用，通常用于伴随逻辑的跟踪。

- **OnApplicationPause**
    ```
    This is called at the end of the frame where the pause is detected, effectively between the 
    normal frame updates. One extra frame will be issued after OnApplicationPause is called to 
    allow the game to show graphics that indicate the paused state.
    ```
    在帧的结尾处调用此函数（在正常帧更新之间有效检测到暂停）。在调用OnApplicationPause之后，将发出一个额外帧，从而允许游戏显示图形来指示暂停状态。

- **OnApplicationQuit**:
    ```
    This function is called on all game objects before the application is quit. 
    In the editor it is called when the user stops playmode.
    ```
    此函数将在应用退出之前调用，例如在Editor中再次点击`Play`按钮关闭播放模式。

- **OnDisable**:
    ```
    This function is called when the behaviour becomes disabled or inactive.
    ```
    此函数将在行为被禁用或处于非活动状态时调用，例如OnApplicationQuit函数调用之后。

- **OnDestroy**:
    ```
    This function is called after all frame updates for the last frame of the object’s existence 
    (the object might be destroyed in response to Object.Destroy or at the closure of a scene).
    ```
    此函数将在对象存在的最后一帧完成所有帧更新之后调用，可能应`Object.Destroy`函数要求或在场景关闭时销毁该对象。

### 函数执行顺序测试

（以下测试**不包含**协程函数）
为一个Cube物体添加Lifecycle脚本组件。

#### 测试代码

```csharp
using UnityEngine;

/// <summary>
/// Simple demo for script lifecycle .
/// <summary>

public class Lifecycle : MonoBehaviour
{
    private void Awake()
    {
        Debug.Log("Function : Awake() " + "Time : " + Time.time + " Object.name : " + this.name);
    }

    private void OnEnable()
    {
        Debug.Log("Function : OnEnable() " + "Time : " + Time.time + " Object.name : " + this.name);
    }

    private void Reset()
    {
        Debug.Log("Function : Reset() " + "Time : " + Time.time + " Object.name : " + this.name);
    }

    private void Start()
    {
        Debug.Log("Function : Start() " + "Time : " + Time.time + " Object.name : " + this.name);
    }

    private void FixedUpdate()
    {
        if (Time.time < 0.1)
        {
            Debug.Log("Function : FixedUpdate() " + "Time : " + Time.time + " Object.name : " + this.name);
        }
    }

    // private void OnMouseDown()
    // {
    //     Debug.Log("Function : OnMouseDown() " + "Time : " + Time.time + " Object.name : " + this.name);
    // }

    private void Update()
    {
        if (Time.time < 0.1)
        {
            Debug.Log("Function : Update() " + "Time : " + Time.time + " Object.name : " + this.name);
        }
    }

    private void LateUpdate()
    {
        if (Time.time < 0.1)
        {
            Debug.Log("Function : LateUpdate() " + "Time : " + Time.time + " Object.name : " + this.name);
        }
    }

    // private void OnGUI()
    // {
    //     if (Time.time < 0.1)
    //     {
    //         Debug.Log("Function : OnGUI() " + "Time : " + Time.time + " Object.name : " + this.name);
    //     }
    // }

    private void OnApplicationPause(bool pause = true)
    {
        if (pause)
        {
            Debug.Log("Function : OnApplicationPause() " + "Time : " + Time.time + " Object.name : " + this.name);
        }
        
    }

    private void OnApplicationQuit()
    {
        Debug.Log("Function : OnApplicationQuit() " + "Time : " + Time.time + " Object.name : " + this.name);
    }

    private void OnDisable()
    {
        Debug.Log("Function : OnDisable() " + "Time : " + Time.time + " Object.name : " + this.name);
    }

    public void OnDestroy()
    {
        Debug.Log("Function : OnDestroy() " + "Time : " + Time.time + " Object.name : " + this.name);
    }
}
```

#### 测试结果


<img src="Unity-Script-Lifecycle/lifecycle_demo.png" alt="lifecycle demo">

#### 测试反馈

控制台输出结果表明，运行场景时在Editor中点击`Pause`按钮将不会调用OnApplicationPause函数。

## 官方文档 

[Order of Execution for Event Functions](https://docs.unity3d.com/Manual/ExecutionOrder.html)
