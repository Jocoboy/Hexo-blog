---
title: Unity即时模式GUI
date: 2020-01-10 19:28:07
categories:
- Unity
- C#
tags:
- Unity-UI
---

```
Unity’s IMGUI controls make use of a special function called OnGUI(). 
The OnGUI() function gets called every frame as long as the containing script is enabled 
- just like the Update() function.
```
Unity的IMGUI控件使用一个名为OnGUI()的特殊函数。只要启用包含的脚本，就会在每帧调用OnGUI()函数，就像Update()函数一样。
<!--more-->

## IMGUI控件

### 控件类型

- **Label**:
    ```
    The Label is non-interactive. It is for display only. It cannot be clicked or otherwise moved. 
    It is best for displaying information only.
    ```
    Label为非交互式控件。此控件仅用于显示目的。不能单击，也不能以其他方式进行移动。此控件最适合于纯粹显示信息之用。

- **Button**:
    ```
    The Button is a typical interactive button. It will respond a single time when clicked, no matter 
    how long the mouse remains depressed. The response occurs as soon as the mouse button is released.
    ```
    Button是典型的交互式按钮。点击按钮时，无论鼠标按下多久，都只会响应一次。松开鼠标按键后会立即响应。

- **RepeatButton**:
    ```
    RepeatButton is a variation of the regular Button. The difference is, RepeatButton will respond 
    every frame that the mouse button remains depressed. This allows you to create 
    click-and-hold functionality.
    ```
    RepeatButton是常规Button的变体。区别在于，RepeatButton将响应鼠标按键保持按下状态的每一帧。由此可以创建单击并保持功能(例如，射击连发功能）。

- **TextField**:
    ```
    The TextField Control is an interactive, editable single-line field containing a text string.
    ```
    TextField控件是一个包含文本字符串的交互式可编辑单行字段。

- **TextArea**:
    ```
    The TextArea Control is an interactive, editable multi-line area containing a text string.
    ```
    TextArea控件是一个包含文本字符串的交互式可编辑多行区域。 

- **Toggle**:
    ```
    The Toggle Control creates a checkbox with a persistent on/off state. 
    The user can change the state by  clicking on it. 
    ```
    Toggle控件创建具有持久开/关状态的复选框。用户可通过点击该复选框来更改状态。

- **Toolbar**:
    ```
    The Toolbar Control is essentially a row of Buttons. Only one of the Buttons on the Toolbar 
    can be active at a time, and it will remain active until a different Button is clicked. 
    This behavior emulates the behavior of a typical Toolbar. You can define an arbitrary 
    number of Buttons on the Toolbar.
    ```
    Toolbar控件本质上是一行Button。在Toolbar上，一次只能有一个Button处于激活状态，并且此Button将一直保持激活状态，直到点击其他Button。此行为模拟典型Toolbar的行为。在Toolbar上可以定义任意数量的Button。

- **SelectionGrid**:
    ```
    The SelectionGrid Control is a multi-row Toolbar. You can determine the 
    number of columns and rows in the grid. Only one Button can be active at time. 
    ```
    SelectionGrid控件是一种多行Toolbar。您可以决定网格中的列数和行数。一次只能激活一个Button。

- **HorizontalSlider**:
    ```  
    The HorizontalSlider Control is a typical horizontal sliding knob that can be 
    dragged to change a value between predetermined min and max values.
    ```
    HorizontalSlider控件是一个典型的水平滑钮，可拖动该滑钮来更改介于预定最小值和最大值之间的值。

- **VerticalSlider**:
    ```
    The VerticalSlider Control is a typical vertical sliding knob that can be 
    dragged to change a value between predetermined min and max values.
    ```
    VerticalSlider控件是一个典型的垂直滑钮，可拖动该滑钮来更改介于预定最小值和最大值之间的值。

- **HorizontalScrollbar**:
    ```
    The HorizontalScrollbar Control is similar to a Slider Control, but visually 
    similar to Scrolling elements for web browsers or word processors. This control 
    is used to navigate the ScrollView Control.
    ```
    HorizontalScrollbar控件类似于Slider控件，但在视觉上类似于Web浏览器或文字处理程序的滚动元素。此控件用于导航 ScrollView 控件。


- **VerticalScrollbar**:
    ```
    The VerticalScrollbar Control is similar to a Slider Control, but visually 
    similar to Scrolling elements for web browsers or word processors. This control 
    is used to navigate the ScrollView Control.
    ```
    VerticalScrollbar控件类似于Slider控件，但在视觉上类似于Web浏览器或文字处理程序的滚动元素。此控件用于导航 ScrollView 控件。

- **ScrollView**:
    ```
    ScrollViews are Controls that display a viewable area of a much larger set of Controls.
    ```
    ScrollView控件可显示一个包含更大控件集合的可视区域。 

- **Window**:
    ```
    Windows are drag-able containers of Controls. They can receive and lose focus when clicked. 
    Because of this, they are implemented slightly differently from the other Controls. 
    Each Window has an id number, and its contents are declared inside a separate function 
    that is called when the Window has focus. 
    ```
    Window是可拖动的控件容器。点击时，Window可获得和失去焦点。因此，实现方式与其他控件略有不同。每个Window都有一个 id编号，并且其内容在一个单独的函数内声明，该函数在Window获得焦点时调用。

### 控件测试

为一个GameObject添加IMGUIControls脚本实例。

```csharp
using UnityEngine;

/// <summary>
///  A simple demo for IMGUI Controls.
/// <summary>
public class IMGUIControls : MonoBehaviour
{

    private string textFieldString = "The TextField Control is an interactive, editable single-line field containing a text string.";
    private string textAreaString = "The TextArea Control is an interactive, editable multi-line area containing a text string.";

    private bool toggleBool = true;

    private int toolbarInt = 0;
    private readonly string[] toolbarStrings = { "Toolbar 1", "Toolbar 2", "Toolbar 3" };

    private int selectionGridInt = 0;
    private readonly string[] selectionStrings = { "Grid 1", "Grid 2", "Grid 3", "Grid 4", "Grid 5" };

    private float hSliderValue = 0.0f;
    private float vSliderValue = 0.0f;

    private float hScrollbarValue;
    private float vScrollbarValue;

    private Vector2 scrollViewVector = Vector2.zero;
    private string innerText = "ScrollViews are Controls that display a viewable area of a much larger set of Controls.";

    private Rect windowRect = new Rect(220, 240, 200, 50);
    private void WindowFunction(int windowID)
    {
        //print("WindowFunction called");
    }
    private void OnGUI()
    {
        GUI.Label(new Rect(10, 10, 200, 20), "Label");

        // In UnityGUI, Buttons will return true when they are clicked. 
        if (GUI.Button(new Rect(10, 40, 200, 50), "Button"))
        {
            print("Button");
        }

        // In UnityGUI, RepeatButtons will return true for every frame that they are clicked.
        if (GUI.RepeatButton(new Rect(10, 110, 200, 50), "RepeatButton"))
        {
            print("RepeatButton");
        }

        // When edits are made to the string, the TextField function will return the edited string.
        textFieldString = GUI.TextField(new Rect(10, 170, 200, 50), textFieldString);
        // When edits are made to the string, the TextArea function will return the edited string.
        textAreaString = GUI.TextArea(new Rect(10, 230, 200, 50), textAreaString);

        // The Toggle function will return a new boolean value if it is clicked. 
        // In order to capture this interactivity, you must assign the boolean to accept the return value of the Toggle function.
        toggleBool = GUI.Toggle(new Rect(10, 280, 200, 30), toggleBool, "Toggle");

        // To make the Toolbar interactive, you must assign the integer to the return value of the function. 
        // The number of elements in the content array that you provide will determine the number of Buttons that are shown in the Toolbar.
        toolbarInt = GUI.Toolbar(new Rect(10, 310, 200, 50), toolbarInt, toolbarStrings);

        // To make the SelectionGrid interactive, you must assign the integer to the return value of the function. 
        // The number of elements in the content array that you provide will determine the number of Buttons that are shown in the SelectionGrid.
        selectionGridInt = GUI.SelectionGrid(new Rect(10, 360, 200, 60), selectionGridInt, selectionStrings, 3);

        hSliderValue = GUI.HorizontalSlider(new Rect(220, 10, 100, 30), hSliderValue, 0.0f, 10.0f);
        vSliderValue = GUI.VerticalSlider(new Rect(220, 30, 100, 30), vSliderValue, 10.0f, 0.0f);

        hScrollbarValue = GUI.HorizontalScrollbar(new Rect(220, 70, 100, 30), hScrollbarValue, 1.0f, 0.0f, 10.0f);
        vScrollbarValue = GUI.VerticalScrollbar(new Rect(220, 90, 100, 30), vScrollbarValue, 1.0f, 10.0f, 0.0f);

        // ScrollViews require two Rects as arguments. 
        // The first Rect defines the location and size of the viewable ScrollView area on the screen. 
        // The second Rect defines the size of the space contained inside the viewable area. 
        // If the space inside the viewable area is larger than the viewable area, Scrollbars will appear as appropriate. 
        // You must also assign and provide a 2D Vector which stores the position of the viewable area that is displayed.
        scrollViewVector = GUI.BeginScrollView(new Rect(220, 130, 250, 100), scrollViewVector, new Rect(0, 0, 400, 400));
        innerText = GUI.TextArea(new Rect(0, 0, 400, 400), innerText);
        GUI.EndScrollView();

        windowRect = GUI.Window(0, windowRect, WindowFunction, "My Window");

        // To detect if the user did any action in the GUI (clicked a button, dragged a slider, etc), read the GUI.changed value from your script. 
        // This gets set to true when the user has done something, making it easy to validate the user input.
        if (GUI.changed)
        {
            print("The toolbar was clicked");

            switch (toolbarInt)
            {
                case 0:
                    print("First button was clicked");
                    break;
                case 1:
                    print("Second button was clicked");
                    break;
                case 2:
                    print("Third button was clicked");
                    break;
            }
        }
    }
}

```

<img src="Unity-UI-IMGUI/controls_demo.png" alt="Controls demo">

## IMGUI布局模式

```
Depending on which Layout Mode you’re using, there are different hooks for controlling where your 
Controls are positioned and how they are grouped together. In Fixed Layout, 
you can put different Controls into Groups. In Automatic Layout, you can put different Controls 
into Areas, Horizontal Groups, and Vertical Groups.
```
根据使用的布局模式，可通过不同的挂钩来控制控件的位置以及控件如何组合在一起。在固定布局中，可将不同的控件放入组中。在自动布局中，可将不同的控件放入区域、水平组和垂直组中。

### 固定布局

- **Fixed Layout - Groups**:
    ```
    Groups are a convention available in Fixed Layout Mode. They allow you to define areas of the 
    screen that contain multiple Controls. You define which Controls are inside a Group by 
    using the GUI.BeginGroup() and GUI.EndGroup() functions. All Controls inside a Group will be positioned 
    based on the Group’s top-left corner instead of the screen’s top-left corner. This way, if you 
    reposition the group at runtime, the relative positions of all Controls in the group will be maintained.
    ```
    组是固定布局模式中的布局规则。使用组可以定义包含多个控件的屏幕区域。为定义组中包含的控件，需要使用 GUI.BeginGroup()和GUI.EndGroup()函数。组内的所有控件将根据组的左上角而不是屏幕的左上角进行定位。因此，如果在运行时重新定位组，则将保持组中所有控件的相对位置。

### 自动布局

- **Automatic Layout - Areas**:
    ```
    Areas are used in Automatic Layout mode only. They are similar to Fixed Layout Groups in functionality, 
    as they define a finite portion of the screen to contain GUILayout Controls. Because of 
    the nature of Automatic Layout, you will nearly always use Areas.

    In Automatic Layout mode, you do not define the area of the screen where the Control will be drawn 
    at the Control level. The Control will automatically be placed at the upper-leftmost point of 
    its containing area. This might be the screen. You can also create manually-positioned Areas. 
    GUILayout Controls inside an area will be placed at the upper-leftmost point of that area.
    ```
    区域仅用于自动布局模式。区域定义了有限的屏幕区域来包含 GUILayout 控件，因此在功能上类似于固定布局组。由于自动布局的性质，几乎始终要用到区域。
    在自动布局模式下，不需要在控制级别定义绘制控件的屏幕区域。控件将自动放置在包含该控件的区域的最左上角。此区域可能是指屏幕。此外也可以创建手动定位的区域。一个区域内的GUILayout控件将放置在该区域的最左上角。

- **Automatic Layout - Horizontal and Vertical Groups**:
    ``` 
    When using Automatic Layout, Controls will by default appear one after another from top to bottom. 
    There are plenty of occasions you will want finer level of control over where your Controls are 
    placed and how they are arranged. If you are using the Automatic Layout mode, you have the 
    option of Horizontal and Vertical Groups.

    Like the other layout Controls, you call separate functions to start or end these groups. The 
    specific functions are GUILayout.BeginHorizontal(), GUILayout.EndHorizontal(), 
    GUILayout.BeginVertical(), and GUILayout.EndVertical().

    Any Controls inside a Horizontal Group will always be laid out horizontally. Any Controls 
    inside a Vertical Group will always be laid out vertically. This sounds plain until 
    you start nesting groups inside each other. This allows you to arrange any number 
    of controls in any imaginable configuration.
   ```
   使用自动布局时，默认情况下控件将从上到下依次出现。在很多情况下，需要更精确控制控件的放置位置以及排列方式。如果使用自动布局模式，则可以选择水平和垂直组。
   与其他布局控件一样，可以调用单独的函数来开始或结束这些组。这些函数为 GUILayout.BeginHoriztontal()、GUILayout.EndHorizontal()、GUILayout.BeginVertical() 和 GUILayout.EndVertical()。
   水平组内的所有控件都将始终采用水平布局方式。垂直组内的所有控件都将始终采用垂直布局方式。这听起来很简单，但若要将组嵌套在彼此内部，就不那么简单了。通过嵌套的方式可在任何能够想象的配置中排列任意数量的控件。

### 布局测试

为一个GameObject添加IMGUILayoutModes脚本实例。

```csharp
using UnityEngine;

/// <summary>
///  A simple demo for IMGUI Layout Modes.
/// <summary>
public class IMGUILayoutModes : MonoBehaviour
{
    public Texture2D bgImage;
    public Texture2D fgImage;
    public float playerEnergy = 1.0f;

    private float sliderValue = 1.0f;
    private float maxSliderValue = 10.0f;

    private void OnGUI()
    {
        // You define which Controls are inside a Group by using the GUI.BeginGroup() and GUI.EndGroup() functions. 
        GUI.BeginGroup(new Rect(Screen.width / 2 - 50, Screen.height / 2 - 50, 100, 100));
        GUI.Box(new Rect(0, 0, 100, 100), "Group is here");
        GUI.Button(new Rect(10, 40, 80, 30), "Click me");
        GUI.EndGroup();

        // You can also nest multiple Groups inside each other. 
        // When you do this, each group has its contents clipped to its parent’s space.
        GUI.BeginGroup(new Rect(Screen.width / 2 -128 ,Screen.height / 2 +116,  256, 32));
        GUI.Box(new Rect(0, 0, 256, 32), bgImage);
            GUI.BeginGroup(new Rect(0, 0, playerEnergy * 256, 32));
            GUI.Box(new Rect(0, 0, 256, 32), fgImage);
            GUI.EndGroup();
        GUI.EndGroup();

        GUILayout.Button("I am not inside an Area");
        GUILayout.BeginArea(new Rect(0, Screen.height / 2, 300, 300));
        // Notice that inside an Area, Controls with visible elements like Buttons and Boxes will stretch their width to the full length of the Area.
        GUILayout.Button("I am a regular Automatic Layout Button");
        // You can use GUILayoutOptions to override some of the Automatic Layout parameters. 
        // You do this by providing the options as the final parameters of the GUILayout Control.
        GUILayout.Button("My width has been overridden", GUILayout.Width(200));
        GUILayout.EndArea();

        // Using nested Horizontal and Vertical Groups.
        GUILayout.BeginArea(new Rect(0, 400, 200, 60));
            GUILayout.BeginHorizontal();
            if (GUILayout.RepeatButton("Increase max\nSlider Value"))
            {
                maxSliderValue += 3.0f * Time.deltaTime;
            }
                GUILayout.BeginVertical();
                GUILayout.Box("Slider Value: " + Mathf.Round(sliderValue));
                sliderValue = GUILayout.HorizontalSlider(sliderValue, 0.0f, maxSliderValue);
                GUILayout.EndVertical();
            GUILayout.EndHorizontal();
        GUILayout.EndArea();
    }
}
```

<img src="Unity-UI-IMGUI/layout_modes_demo.png" alt="LayoutModes demo">

## 官方文档

[Immediate Mode GUI (IMGUI)](https://docs.unity3d.com/Manual/GUIScriptingGuide.html)