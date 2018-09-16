---
title: 1.5 OpenGL坐标系统
date: 2018-09-16 20:31:50
tags: 
  - OpenGL ES
categories: 
  - OpenGL ES
---

1.5 OpenGL坐标系统
================

[TOC]

## 一、概述

我们曾经在《1.1 基本概念介绍》里，简单地展示过OpenGL里的坐标转换过程（这张图将会在后边被多次引用）：

![OpenGLCoordinateSystem](http://ovl0w8qm7.bkt.clouddn.com/opengl_coordinate_systems.png)

在Vertex Shader运行后，我们可见的所有顶点都会转变成标准化设备坐标（Normal Device Coordinate，简称NDC），换句话说，每个顶点的x，y，z坐标都在[-1, 1]区间内，不在这个区间内的坐标都将不可见。我们一般只需要指定一个坐标范围，然后在Vertex Shader中将范围内的坐标转换为NDC，最后这些NDC坐标在OpenGL中会被传给光栅器（Rasterizer），最终变为屏幕上的二维坐标，也就是像素。

如图所示，将坐标转换为NDC，接着再转换成屏幕坐标的过程是分步进行的，类似于流水线（Pipeline）一样。将这个过程拆分成多个坐标系的转换过程有一个好，一些操作或者说变换在特定的坐标系里会更加易于进行。这些坐标系分别是：

- Local Space（Object Space），本地坐标空间
- World Space，世界坐标空间
- View Space（Eye Space），观察坐标空间
- Clip Space
- Screen Space

在Vertex Shader中，一般顶点坐标是这样计算的：

```c
layout (location = 0) in vec3 aPos;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main() {
	gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

可以看出，所有顶点是依次经过model，view和projection矩阵变换最后得到NDC坐标的。

今天我们就详细讲讲这些坐标系统。



## 二、 坐标系统

### 1. Local Space

本地坐标空间是指创建一个空间模型时所用的坐标空间。大部分情况下，我们建立一个模型的时候，会默认以（0，0，0）为原点来建模。即便是做一个大工程，需要用到非常多的模型，我们也是从头开始，在本地坐标空间里把模型逐个搭建好，再在后续的坐标空间里进行整合。

以建立立方体模型为例，顶点坐标设置如下：

```java
float vertices[] = { // 一个立方体六个面，每个面由两个三角形组成，每个三角形三个点，共36个点
		-0.5f, -0.5f, -0.5f,
		0.5f, -0.5f, -0.5f,
		0.5f,  0.5f, -0.5f,
		0.5f,  0.5f, -0.5f,
		-0.5f,  0.5f, -0.5f,
		-0.5f, -0.5f, -0.5f,

		-0.5f, -0.5f,  0.5f,
		0.5f, -0.5f,  0.5f,
		0.5f,  0.5f,  0.5f,
		0.5f,  0.5f,  0.5f,
		-0.5f,  0.5f,  0.5f,
		-0.5f, -0.5f,  0.5f,

		-0.5f,  0.5f,  0.5f,
		-0.5f,  0.5f, -0.5f,
		-0.5f, -0.5f, -0.5f,
		-0.5f, -0.5f, -0.5f,
		-0.5f, -0.5f,  0.5f,
		-0.5f,  0.5f,  0.5f,

		0.5f,  0.5f,  0.5f,
		0.5f,  0.5f, -0.5f,
		0.5f, -0.5f, -0.5f,
		0.5f, -0.5f, -0.5f,
		0.5f, -0.5f,  0.5f,
		0.5f,  0.5f,  0.5f,

		-0.5f, -0.5f, -0.5f,
		0.5f, -0.5f, -0.5f,
		0.5f, -0.5f,  0.5f,
		0.5f, -0.5f,  0.5f,
		-0.5f, -0.5f,  0.5f,
		-0.5f, -0.5f, -0.5f,

		-0.5f,  0.5f, -0.5f,
		0.5f,  0.5f, -0.5f,
		0.5f,  0.5f,  0.5f,
		0.5f,  0.5f,  0.5f,
		-0.5f,  0.5f,  0.5f,
		-0.5f,  0.5f, -0.5f,
	};
```

### 2. World Space

上一小节我们提出了在Local Space中建立模型，也就是说，基本上我们所有模型都是基于（0，0，0）原点建立的，如果直接把这些模型都绘制出来的话，可能最终效果我们会看到所有这些模型都堆在一个地方重叠了，这显然不是我们想要的效果。为了将每个模型放到它应在的位置，我们引入了世界坐标空间的概念。我们为某个模型定义一个偏移量（offset），例如说在X轴上偏移2，在Y轴上偏移5，在Z轴上偏移-15，也就是把这个模型做一个（2， 5，-15）的平移操作，平移后这个模型到了它应在的位置，这时候，我们可以认为它的顶点坐标是属于世界坐标空间的。

![OpenGLCoordinateSystem](http://ovl0w8qm7.bkt.clouddn.com/opengl_coordinate_systems.png)

让我们回顾一下这张图，刚刚说的平移操作其实就是model矩阵的变换结果，我们通过model矩阵，把某个模型的所有顶点统一地移动到了某个位置，这样我们就把模型顶点坐标从Local Space变换到World Space了。

Model变换矩阵不仅仅可以完成平移操作，还可以完成缩放、和旋转的操作。例如说我们想画多个立方体，这些立方体零散分布在空间中，这时其实就是在逐个绘制立方体时，给每个立方体一个model矩阵，每个model矩阵都有自己的平移、旋转、缩放参数，把model矩阵的值赋给Vertex Shader，这样最终绘制出来的效果就是每个立方体都在不同的位置，大小、方向各不相同，这就是世界空间坐标以及model矩阵变换的意义。

### 3. View Space

观察坐标空间，在OpenGL中通常会理解为**相机空间**或者**视觉空间**。View Space是将World Space坐标转换成用户视角上的坐标。不妨想象一下在一个世界坐标空间内，我们放置了许多立方体，我们可以从一个屏幕上观察这些立方体，而屏幕内容就是来自一个摄像机（Camera），随着摄像机的移动，视角调整，我们可以从不同角度、不同方向观察这些立方体。

反过来看，我们移动摄像机，其实就相当于我们自身不动，移动的是整个世界坐标空间，通过矩阵变换，我们让特定的一些顶点靠近Camera，一些顶点远离Camera，这样，就实现了我们想要的调整视角、视距来观察世界的效果。

![OpenGLCoordinateSystem](http://ovl0w8qm7.bkt.clouddn.com/opengl_coordinate_systems.png)

从图中可以看出，这些变换会存储在View Matrix中，View Matrix会将顶点坐标从World Space变换到View Space中。关于这部分我们会在下一篇文章里更加详细地展开讲解。

### 4. Clip Space

当所有Vertex Shader运行完之后，OpenGL希望大部分坐标会落在一个特定的范围里，而不在这个范围内的坐标都会被裁剪掉（Clipped），这就是Clip Space命名缘由。裁剪多余的坐标后，剩余的坐标就会形成Fragment，也就是光栅化，最后出现在屏幕上。

跟之前的类似，我们会定义一个Projection Matrix来完成从View Space到Clip Space的坐标变换。在Projection Matrix里，我们指定了坐标的范围，例如[-1000, 1000]，接下来，我们定义的坐标范围在[-1000, 1000]内的顶点都会被按比例映射到[-1, 1]区间内；而在[-1000, 1000]区间外的顶点都会映射到[-1, 1]区间外，最终会被裁剪掉。

如果我们绘制的是一个图元（primitive），例如说一个三角形，这个三角形的顶点坐标跨度比较大，超出了我们定义的坐标范围的话，那么OpenGL会自动重新构造一个或者多个三角形（最终顶点范围都在[-1, 1]内）来适应坐标范围。

形象地说，Projection Matrix会创建一个视锥（Frustum），所有在视锥里的坐标点都会出现在屏幕上。这种将特定范围内的坐标映射到NDC坐标中的过程称为**投影**。当所有顶点坐标都转换到Clip Space之后，我们会把顶点坐标的x，y，z分量都除以齐次坐标的w分量，得到其原始坐标，这个过程叫作**透视除法**，了解一下。这一步会在Vertex Shader执行的最后阶段里自动执行。

在该阶段过后，我们的坐标就会映射为屏幕坐标（通过glViewport方法设置），并光栅化为Fragment输出。

根据表现形式的不同，投影一般分为两种，一种是正射投影（Orthographic Projection），另一种是透视投影（Perspective Projection），两种投影分别会建立不同的视锥体（Frustum），视觉效果的差别也会因为视锥体的不同而差异化。

#### 4.1 正射投影 Orthographic Projection

正射投影定义了一个长方体样式的视锥体，不在该视锥体范围内的顶点都会被裁剪掉（如图）。

![Orthographic Projection Frustum](https://learnopengl.com/img/getting-started/orthographic_frustum.png)



当创建正射投影矩阵时，我们需要指定视锥体的长、宽、高，具体来说就是X轴的坐标范围、Y轴的坐标范围以及近平面、远平面的距离。我们可以使用GLM的函数来获取正射投影矩阵：

```c++
glm::mat4 projection = glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, 0.1f, 100.0f);
```

这样我们就定义了视锥体left为0，right为800，bottom为0，top为600，近平面距离0.1，远平面距离100f的长方体。

正射投影的思想非常直观，但是会造成不真实的视觉效果，因为它没有将**透视**效果考虑进去。正射投影跟透视投影的区别可以通过下图来辨别：

![Differences between orthographic projection and perspective projection](https://learnopengl.com/img/getting-started/perspective_orthographic.png)

#### 4.2 透视投影 Perspective Projection

透视投影其实已经在前边的文章里讲过了，这里简单复习一下。在现实生活中，我们知道远处的东西看起来更小，而近处的东西看起来更大；两条无限延伸、互相平行的直线会在地平线处汇聚在一起。这种奇妙的效果我们称之为**透视**。

透视投影矩阵会定义一个四棱锥柱样式的视锥体，除此之外还会修改每个顶点的w分量，距离Camera越远的顶点坐标的w分量就越大，被变换到Clip Space的坐标都会落到[-w, w]区间内，不在该区间内的顶点都会被裁剪掉。而因为OpenGL要求最终坐标需要在[-1, 1]区间内，所以所有坐标的所有分量都会除以w分量（透视除法），得到最终在[-1, 1]区间内的坐标。当w分量越大的时候，做透视除法后的坐标值就越小，这样物体在视觉上就越小，这也就是我们需要的“透视”效果。

我们可以通过GLM的内置函数建立透视投影矩阵：

```c++
glm::mat4 projection = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
```

这里我们指定了fov（Field of View，视角大小）为45度（转为弧度），宽高比，近平面距离0.1f，远平面距离100f的一个视锥体：

![Perspective Projection Frustum](https://learnopengl.com/img/getting-started/perspective_frustum.png)

近平面距离一般设置为0.1f，远平面距离一般设置为100f。

至于如何根据这些参数来计算透视投影矩阵，请参阅《1.4 线代基础与矩阵变换原理》。

### 5. Screen Space

在进行透视除法后，我们得到了NDC坐标，剩下的就交给OpenGL自行把NDC坐标转换成屏幕像素坐标，也就是把NDC坐标映射到Screen Space当中。在进行最后一步之前，我们需要通过`glViewport(x, y, width, height)`来告诉OpenGL我们展示用的窗口大小是多少，绘制时的横纵轴偏移值x，y分别是多少。

最终坐标的形式如下：

$\begin{bmatrix}x_w \\\ y_w \\\ z_w\end{bmatrix} = \begin{bmatrix}\frac{width}{2}x_{ndc} + x + \frac{width}{2} \\\ \frac{height}{2}y_{ndc} + y + \frac{height}{2} \\\ \frac{far - near}{2}z_{ndc} + \frac{far + near}{2}\end{bmatrix}$

可以通过结果推导出Viewport Matrix：

$\begin{bmatrix}\frac{width}{2} & 0 & 0 & x + \frac{width}{2} \\\ 0 & \frac{height}{2} & 0 & y + \frac{height}{2} \\\ 0 & 0 & \frac{far - near}{2} & \frac{far + near}{2} \\\ 0 & 0& 0& 1 \end{bmatrix}$

大家可以画图比划一下。

## 三、总结

我们来结合下图总结一下OpenGL中坐标的变换过程。

![OpenGLPipeline](http://ovl0w8qm7.bkt.clouddn.com/OpenGLpipeline.png)

首先，我们在Local Space里建立好模型，得到一系列model coordinates顶点，通过将这些顶点坐标左乘Model Matrix，我们对模型进行了平移、旋转和缩放，得到了world coordinates顶点；接下来，我们调整Camera，观察我们所需要的视角，这一步通过讲顶点坐标左乘View Matrix完成，得到eye coordinates顶点；然后，我们根据实际需要对模型进行投影变换（正射投影或者透视投影），讲顶点坐标左乘Projection Matrix得到clip coordinates，紧接着我们对齐次坐标形式的顶点的x, y, z, w分量都除以w分量，也就是做透视除法，得到NDC坐标；最后，我们通过`glViewport(x, y, width, height);`方法告诉OpenGL绘制窗口的大小，让OpenGL自行将NDC坐标转换成屏幕坐标，最后在屏幕上渲染出我们想要的图像。
上述local coordinates到clip coordinates的过程可以用这么一条式子表达：
$V_{clip} = M_{projection} \cdot M_{view} \cdot M_{model} \cdot V_{local}$

需要注意的是在OpenGL里，向量跟矩阵相乘是从右往左计算的，所以这里的计算过程实际上是$M_{model}$先与$V_{local}$相乘，计算结果再跟$M_{view}$相乘，最后再跟$M_{projection}$相乘。

至此，OpenGL从原始顶点数据到最终在屏幕上渲染出来的流水线过程讲诉完毕。