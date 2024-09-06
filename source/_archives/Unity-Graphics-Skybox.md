---
title: Unity自定义天空盒
date: 2020-01-06 21:02:25
categories:
- Unity
- C#
tags:
- Unity-Graphics
---

```
A Skybox is a six-sided cube that Unity draws behind all graphics in the Scene. 
```
一个天空盒是一个六面立方体，会将游戏场景中的所有图形包裹。

<!--more-->
创建步骤
## 自定义天空盒

### 6-sided

#### 准备

6张TIFF/TIF格式(具有跨平台性)的方位局部图，1024×1024px。

#### 创建步骤

1. 创建与天空盒六个面相对应的六个纹理，将它们放在项目的`Assets`文件夹中。
```
Make six Textures that correspond to each of the six sides of the skybox, and put them into your
Project’s Assets folder.
```
<img src="Unity-Graphics-Skybox/skybox_textures.png" alt="skybox textures">

2. 对于每个纹理，需要将包裹模式从`Repeat`更改为`Clamp`。如果不这样做，边缘上的颜色将不匹配。
```
For each Texture, you need to change the wrap mode from Repeat to Clamp.
If you don’t do this, colors on the edges do not match up.
```
<img src="Unity-Graphics-Skybox/wrap_mode.png" alt="wrap mode">

3. 从菜单栏中选择`Assets > Create > Material`以创建新材质。 
```
Create a new Material. To do this, choose Assets > Create > Material from the menu bar.
```

4. 在`Inspector`面板的顶部选择`Shader`下拉选单，然后选择`Skybox/6 Sided`。
```
Select the Shader drop-down and choose Skybox/6 Sided.
```

5. 将6个纹理分配给材质中的每个纹理字段。为此，可将每个纹理从`Project`面板拖放到相应的字段上。
```
Assign the six Textures to each Texture slot in the Material. 
To do this, you can drag each Texture from the Project View onto the corresponding slots.
```
<img src="Unity-Graphics-Skybox/skybox_inspector_01.png" alt="skybox inspector 01">

6. 最后，将天空盒分配给当前场景，须执行以下操作：
    - 在菜单栏中选择`Window > Rendering > Lighting Settings`。
    - 在随后出现的窗口中选择Scene选项卡。
    - 将新的天空盒材质拖放到`Skybox`字段。
```
To assign the skybox to the Scene you’re working on:

- From the menu bar, choose Window > Rendering > Lighting Settings.
- In the window that appears, select the Scene tab.
- Drag the new Skybox Material to the Skybox slot.
```

<img src="Unity-Graphics-Skybox/skybox_application.png" alt="skybox application">

### panoramic

#### 准备

1张png格式的（360°/720°）全景图，2048×1024px/4096×2048px。

<img src="Unity-Graphics-Skybox/skybox_panoramic.png" alt="skybox panoramic">

#### 创建步骤

步骤1~3及步骤6同6-sided。

4. 在`Inspector`面板的顶部选择`Shader`下拉选单，然后选择`Skybox/Panoramic`。
```
Select the Shader drop-down and choose Skybox/Panoramic.
```

5. 为材质选择一个球状纹理。
```
Assign a Spherical Texture to the Material. 
```
<img src="Unity-Graphics-Skybox/skybox_inspector_02.png" alt="skybox inspector 02">


## 官方文档

[How do I Make a Skybox?](https://docs.unity3d.com/Manual/HOWTO-UseSkybox.html)