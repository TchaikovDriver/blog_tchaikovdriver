---
title: 1.1 OpenGL基本概念介绍
date: 2018-09-16 19:45:01
tags: 
  - OpenGL ES
categories: 
  - OpenGL ES
---

1.1 基本概念介绍
=====
[TOC]

## 一、什么是OpenGL ES？
OpenGL(Open Graphics Library) 是一套标准的用于渲染2D、3D矢量图形的跨语言、跨平台的应用程序编程接口（API）。而OpenGL ES(Open Graphics Library for Embedded Systems)，顾名思义，是为嵌入式系统（如移动设备的系统）特殊定制的API，去除了glBegin/glEnd，四边形（GL_QUADS）、多边形（GL_POLYGONS）等复杂图元等许多非绝对必要的特性。本次学习的OpenGL ES版本为2.0，是基于OpenGL 3.0裁剪定制而成的。

### 1. OpenGL功能概述
- 图形基本组成要素（What）：Point, Edge, Polygon
- 属性Attribute（How）
- Transformation - Viewing and Modeling
  从运行过程来看，OpenGL其实是一个有限状态机（finite state machine）

### 2. OpenGL函数命名规则
![OpenGLFunctionNaming](http://ovl0w8qm7.bkt.clouddn.com/OpenGLFunctionNaming.jpg)

### 3. OpenGL应用程序的运作流程
![OpenGLProgramBaseStructure](http://ovl0w8qm7.bkt.clouddn.com/OpenGLProgramBaseStructure.png)
一个交互式的OpenGL应用程序一般流程如下：
- 配置并创建一个窗口，用于OpenGL输出
- 初始化一些在整个应用程序运行期间所需的OpenGL状态值
- 处理用户事件，如键盘输入、触摸事件、鼠标移动或者改变窗口大小等
- 根据用户的输入，改变OpenGL状态值，在窗口中绘制图像

可以看出，OpenGL是非面向对象而是面向过程的，开发者需要维护一个状态机，根据用户的输入而改变OpenGL的状态，状态的变更会触发新图像的绘制，最终通过无限循环来实现交互效果。

## 二、OpenGL基本概念
在OpenGL ES中，由于去掉了多边形这种复杂图元，所以只剩下点、线和三角形这三类图元。下面将围绕这三类图元讲诉OpenGL中的一些基本概念。
### 1. Vertex
Vertex就是顶点的意思，一切图形都有Vertex，Vertex序列可以围成一个图形。
### 2. Fragment & Rasterization
光栅化（Rasterization）是指将点、线、三角形映射到屏幕上的像素点的过程，每个映射区域叫作Fragment，换句话说，光栅化就是生成Fragment的过程。

![RasterizationAndFragment](http://ovl0w8qm7.bkt.clouddn.com/rasterization_generate_fragments.png)

### 3. Shader
着色器程序（Shader）用于描述如何绘制（渲染），OpenGL包含了GLSL（OpenGL ShadingLanguage），是一门编程语言，语法与C语言类似。OpenGL渲染需要两种Shader: Vertex Shader和Fragment Shader。
Shader的最终目的就是确定图形的Vertex坐标和Fragment颜色，我们想要用 OpenGL 实现任何效果，无论是静止的光影、色彩、形状，还是运动的物理效果、粒子效果，归根结底都是要根据时间和位置确定Vertex坐标和Fragment颜色。

下面是绘制一个红色三角形所需要的最简单的Vertex Shader和Fragment Shader的代码示例

```c
// Vertex Shader
attribute vec4 aPosition; // 存储了三角形三个点的向量
void main() {
  gl_Position = aPosition;
}

// Fragment Shader
precision mediump float; // 告诉GPU在浮点数计算时使用中等精度就可以了
void main() {
  gl_FragColor = vec4(1, 0, 0, 1); // R G B A, 红色三角形
}
```
### 4. 坐标系

![OpenGLCoordinateSystem](http://ovl0w8qm7.bkt.clouddn.com/opengl_coordinate_systems.png)

下面这张图展示了 OpenGL 通常的处理流程中各个环节的坐标系，以及坐标系之间的转换操作：

- Local space：我们为每个物体建好模型的时候，它们的坐标就是 Local space 坐标；
- World space：当我们要绘制多个物体时，如果直接使用 Local space 的坐标（把所有物体的原点放在一起），它们可能会发生重叠，因此我们需要把它们进行合理的移动、排布，最终各自的坐标就是 World space 的坐标了；
- Model matrix：把 Local space 坐标转换到 World space 坐标所使用的变换矩阵，它是针对每个物体做不同的变换；
- View space：通常也叫 Camera space 或者 Eye space，是从观察者（也就是我们自己）所在的位置出发，所看到的空间；
- View matrix：把 World space 坐标转换到 View space 坐标所使用的变换矩阵，它相当于是在移动相机位置，实际上是反方向移动整个场景（所有物体）；
- Clip space：OpenGL 只会渲染坐标值范围在 `[-1, 1]` 的内容，超出这个范围的内容都会被裁剪掉，这个范围的空间就叫 Clip space，Clip space 的坐标系也叫 Normalized Device Coordinate（NDC）；
- Projection matrix：把 View space 坐标转换到 Clip space 坐标所使用的变换矩阵，它会指定一个可见的范围，只有这个范围内的点才会转换到 NDC 中，而这个范围被称作视锥（Frustum）；projection matrix 有三种创建方式：正投影（Orthographic projection），透视投影（Perspective projection），以及 3D 投影（3D projection）；前两种比较常用；
- Screen space：屏幕上的空间，`glViewport` 调用指定的区域；
- Viewport transform：这一步是 OpenGL 自动完成的，把 Clip space 坐标转换到 Screen space 坐标；