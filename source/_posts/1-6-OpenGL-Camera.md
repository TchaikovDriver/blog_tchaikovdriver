---
title: 1.6 OpenGL Camera
date: 2018-09-16 20:32:51
tags: 
  - OpenGL ES
categories: 
  - OpenGL ES
---

1.6 OpenGL Camera
==========

[TOC]

## 一、前言

在《1.6 OpenGL坐标系统》里，我曾经给出下集预告，说将会在下一篇文章里详细讲解Camera部分，刘老师从不食言，今天，我们就来研究一下Camera。

我们再来看一次这张图：

![OpenGLCoordinateSystem](http://ovl0w8qm7.bkt.clouddn.com/opengl_coordinate_systems.png)

在2.WORLD SPACE中，我们把Camera从$Y$轴正方向朝向$Y$轴负方向，这样取的景就是我们布置的模型世界的俯视图，这样摆放Camera就相当于给出一个View Matrix，使得顶点乘以这个矩阵后，能变换到俯视图里的位置。

所以，本文的主题就是如何设置我们的View Matrix。

## 二、相机Camera

### 1. 确定相机位置与方向

我们先来思考一下，在三维空间里，我们怎么确立一个相机？显然，我们首先要确定位置（position），确定位置以后，我们还要确定方向（direction），也就是视角。位置可以用坐标点来表示，而方向则可以用空间内任意一个坐标点跟位置坐标点联合表示（即向量）：

$position = (x_p, y_p, z_p)$

$target = (x_t, y_t, z_t)$

我们的方向direction就可以用向量来表示：

$\vec{direction} = target - position = (x_t - x_p, y_t - y_p, z_t - z_p)$

这时候我们就碰到一个问题：我们怎么确定相机的正反？想象一下，我们现在拿着相机对着某处拍照，正常情况下我们是正着拿相机（顶朝上），拍摄出来的照片自然是正的；当我们把相机反过来（顶朝下），拍照出来的照片就相当于做了180度旋转一样。因此，我们还需要额外的参数来确定相机的“正反”。

除了方向向量以外，我们还需要指定相机顶部朝上和正右侧的向量来完全确定相机的“正反”。参照下图里的最后一张图，我们最终需要3个互相垂直的向量来确定相机的位置和方向：

![CameraVector](https://learnopengl.com/img/getting-started/camera_axes.png)

首先我们确定了$position = (0, 0, 2)$，随后我们的朝向原点拍摄，$target = (0, 0, 0)$，这时候我们就先确立了第一个方向向量$direction = position - target = (0, 0, 2)$，注意这里的$direction$跟拍摄方向是相反的，后边会解释原因，为了方便后边的计算，我们把$direction$向量标准化为单位向量（模为1的向量）：

$direction = \frac{direction}{||direction||} = \frac{(0, 0, 2)}{2} = (0, 0, 1)$

接着，我们需要确定相机的$right$向量，以表示相机的右侧，这里会用到向量叉积的特性，这里我们先简单地回顾一下向量叉积和右手定则：

![右手定则](http://ovl0w8qm7.bkt.clouddn.com/right_hand_principle.jpg)

我们做叉积$\vec{C} = \vec{A} \times \vec{B}$的时候，首先要确定向量的方向，我们将大拇指以外的四根手指指向$\vec{A}$，然后四指向$\vec{B}$向量方向弯曲，此时，大拇指指向的方向就是向量叉积得到的新向量$\vec{C}$的方向。更多关于向量叉积的内容请参阅《1.4 线性代数基础与矩阵变换原理》。

回到我们的Camera，我们要求相机的右方向，也就是$\vec{right}$，而我们已经有了$\vec{direction}$向量了，利用叉积的性质，我们定义一个$\vec{up} = (0, 1, 0)$，也就是垂直于$X$-$Z$平面的向量，我们用这两个向量做叉积，来求$\vec{right}$向量：

$\vec{right} = \vec{up} \times \vec{direction} = (1, 0, 0)$

可以看出，这里的$\vec{right}$碰巧是$X轴$的正方向的单位向量，碰巧而已，不要在意。

这下我们只剩下相机的顶部朝向，也就是$\vec{top}$向量了，聪明的各位应该能想到，我们同样用向量的叉积来求$\vec{top}$：

$\vec{top} = \vec{direction} \times \vec{right} = (0, 1, 0)$

这下，我们就得到了能确定相机位置、方向的三个单位向量：

$\vec{diretion} = (0, 0, 1)$

$\vec{right} = (1, 0, 0)$ 

$\vec{top} = (0, 1, 0)$

这里需要特别注意的是，$\vec{direction}, \vec{right}, \vec{top}$这三个向量必须是单位向量，也就是向量的模必须为1，如果模不为1的话，需要将各个分量都除以该向量的模来得到单位向量。

### 2. 计算View Matrix

既然知道了相机三个方向的向量，我们就可以开始着手计算View Matrix了。

在这之前，我们要了解一下两个几何概念：

- 标准正交基（Orthonormal Basis）

  在欧几里德空间$R^3$中，向量$\vec{v_1} = (1, 0, 0),  \vec{v_2} = (0, 1, 0),  \vec{v_3} =(0, 0, 1)$是一组标准正交基，这三个**基向量**的线性组合能展开整个三维欧几里德空间。可以认为，一个坐标系统可以由一组标准正交基来定义。

- 右手坐标系

![right_hand_coordinate](http://ovl0w8qm7.bkt.clouddn.com/right_hand_coordinate.jpg)



接下来，我们研究一下相机调整视角的几何意义。

![OpenGLCoordinateSystem](http://ovl0w8qm7.bkt.clouddn.com/opengl_coordinate_systems.png)

我们可以参考上图，当我们把相机在$Y$轴正方向垂直向负方向拍摄时，我们能看到世界的俯视图。这个过程实际上是把World Space里的坐标变换到View Space里，这个变换的几何意义就是把顶点坐标从以$(1, 0, 0), (0, 1, 0), (0, 0, 1)$为标准正交基的坐标系变换到以$\vec{right}, \vec{top}, \vec{direction}$为标准正交基的坐标系（这也是为什么上边我们要求三个向量都必须为单位向量）。

那么，我们这个坐标系转换要怎么做呢？

---

#### 坐标系变换

我们从线性代数的角度来解答这个问题。假设我们有一个父坐标空间$P$，其标准正交基为$\vec{v_1}, \vec{v_2}, \vec{v_3}$，有一个子坐标空间$C$，其标准正交基为$\vec{u_1}, \vec{u_2}, \vec{u_3}$。我们可以用父坐标空间的标准正交基的线性组合来表示子坐标空间的标准正交基：

$\vec{u_1} = a_{11}\vec{v_1} + a_{12}\vec{v_2} + a_{13}\vec{v_3}$

$\vec{u_2} = a_{21}\vec{v_1} + a_{22}\vec{v_2} + a_{23}\vec{v_3}$

$\vec{u_3} = a_{31}\vec{v_1} + a_{32}\vec{v_2} + a_{33}\vec{v_3}$

这个线性组合可以用矩阵来表示：

$\begin{bmatrix}\vec{u_1} \\\ \vec{u_2} \\\ \vec{u_3}\end{bmatrix} = M_{camera} \cdot \begin{bmatrix}\vec{v_1} \\\ \vec{v_2} \\\ \vec{v_3}\end{bmatrix} = \begin{bmatrix}a_{11} & a_{12}  & a_{13} \\\ a_{21} & a_{22}  & a_{23} \\\ a_{31} & a_{32}  & a_{33} \end{bmatrix} \cdot \begin{bmatrix}\vec{v_1} \\\ \vec{v_2} \\\ \vec{v_3}\end{bmatrix}$

 通过这个矩阵$M_{camera}$，我们能把坐标基从父坐标空间变换到子坐标空间，同理，我们的顶点也可以通过跟这个矩阵相乘来实现坐标系的变换（因为每个顶点都是标准正交基的线性组合）。

---

现在我们知道了坐标系要怎么变换了，我们回到计算View Matrix环节。

由于我们World Space的标准正交基是$(1, 0, 0), (0, 1, 0), (0, 0, 1)$，而View Space的标准正交基是$\vec{right}, \vec{top}, \vec{direction}$，所以我们的坐标系变换矩阵就是

$M_{camera} = \begin{bmatrix}\vec{right}_x & \vec{right}_y & \vec{right}_z \\\ \vec{top}_x & \vec{top}_y & \vec{top}_z \\\ \vec{direction}_x & \vec{direction}_y & \vec{direction}_z \end{bmatrix}$

转成齐次坐标可用的形式就是

$M_{camera} = \begin{bmatrix}\vec{right}_x & \vec{right}_y & \vec{right}_z & 0 \\\ \vec{top}_x & \vec{top}_y & \vec{top}_z & 0 \\\ \vec{direction}_x & \vec{direction}_y & \vec{direction}_z & 0 \\\ 0 & 0 & 0 & 1 \end{bmatrix}$

有人可能会问，为什么是$right, top, direction$这个顺序排列？因为我们的父坐标空间跟子坐标空间都是右手坐标系，标准正交基的排列顺序跟坐标点的x, y, z分量排列顺序一致。

通过矩阵$M_{camera}$，我们可以实现顶点坐标系的切换，相当于调整了视角。但是，我们还忽略了另一个关键因素——视距。视距怎么体现？我们可以想象，在World Space中，我们有一个在$Z$正半轴上的相机，相机朝向$Z$负半轴拍摄，当我们把相机向后移动（$Z$坐标增大），我们会感觉世界“远去”；当我们把相机向前移动（$Z$坐标减少），我们会感觉世界“靠近”。没错，其实相机的位置$position(x, y, z)$相当于将World Space坐标向反方向平移。

在《1.4 线性代数基础与矩阵变换原理》里，我们曾经介绍过平移矩阵，结合$position(x, y, z)$我们可以写出平移矩阵：

$M_{translation} = \begin{bmatrix}1 & 0 & 0 & -position_x \\\ 0 & 1 & 0 & -position_y \\\ 0 & 0 & 1 & -position_z \\\ 0 & 0 & 0 & 1\end{bmatrix}$

因为$M_{camera}$和$M_{translation}$都是线性变换，我们可以通过矩阵相乘来把这坐标系变换、平移变换组合在一起，组合结果就是我们想要的View Matrix：

$M_{view} = M_{camera} \cdot M_{translation} =  \begin{bmatrix}\vec{R}_x & \vec{R}_y & \vec{R}_z & 0 \\\ \vec{T}_x & \vec{T}_y & \vec{T}_z & 0 \\\ \vec{D}_x & \vec{D}_y & \vec{D}_z & 0 \\\ 0 & 0 & 0 & 1 \end{bmatrix} \cdot \begin{bmatrix}1 & 0 & 0 & -P_x \\\ 0 & 1 & 0 & -P_y \\\ 0 & 0 & 1 & -P_z \\\ 0 & 0 & 0 & 1\end{bmatrix} $

其中$\vec{R} = \vec{right}, \vec{T} = \vec{top}, \vec{D} = \vec{direction}, \vec{P} = \vec{position}$




## 三、OpenGL计算View Matrix
前边我们已经讲了View Matrix的推导过程，那么在实际OpenGL开发中，我们是怎么获取View Matrix呢？

GLM提供了`glm::lookAt(glm::vec3& eye, glm::vec3& center, glm::vec3& up)`函数供我们直接获取View Matrix，其中`eye`参数就是相机的$position$；`center`是相机的$target$，即相机注视的点，用于计算$\vec{direction}$；`up`参数，用于计算$\vec{right}$。

在上边的例子里，我们可以这样获取View Matrix：

```c++
glm::mat4 viewMatrix = glm::lookAt(
    glm::vec3(0.0f, 0.0f 2.0f),
    glm::vec3(0.0f, 0.0f, 0.0f),
    glm::vec3(0.0f, 1.0f, 0.0f)
);
```
## 四、总结

至此，我们的OpenGL入门系列已经走到尾声，看完OpenGL入门系列文章，相信大家对OpenGL的基础概念以及原理都会有一定程度的了解。往后就是OpenGL进阶系列，在进阶系列出来之前，大家可以去https://learnopengl.com/Getting-started/Creating-a-window 或者 https://learnopengl-cn.github.io/01%20Getting%20started/02%20Creating%20a%20window/ 加深对OpenGL的认识和了解。