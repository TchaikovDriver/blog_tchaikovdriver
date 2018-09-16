---
title: 1.4 OpenGL线性代数基础与矩阵变换原理
date: 2018-09-16 20:30:45
tags: 
  - OpenGL ES
categories: 
  - OpenGL ES
---

1.4 线代基础与矩阵变换原理
=====
[TOC]

## 一、前言
撰写本文的主要目的是讲述计算机图形学中，向量和点通过矩阵运算来实现平移、缩放、旋转以及透视投影的原理。虽然文章主标题是"OpenGL入门"，但是从范畴上来说，这些原理并不局限于OpenGL，因为这些都是计算机图形学的内容，无论是在OpenGL，DirectX还是Vulkan，这些原理都是通用的。
本文首先会讲一些前提背景知识，如一些线性代数的基础，接下来会依次讲解平移、缩放、旋转和透视投影的原理，这些原理都是基于前面说到的线性代数的基础，而且这些基础不难理解，大家看的时候毋须有畏难心理。
对于线性代数零基础的同学来说，如果碰到看不懂的地方，可以反复看第一小节的线性代数基础，同时可以作图辅助理解，再有问题也可以来找刘老师一起探讨。

## 二、数学基础
### 1. 向量

#### 1.1什么是向量？

在计算机图形学中，向量（Vector）是最基本的运算元素之一。

向量是指具有大小（长度）和方向的量，印刷体一般用粗体表示，而手写体一般在向量名上方加箭头表示。如图所示，向量$\vec{BA}$的计算方法就是用A点坐标减去B点坐标，向量$\vec{BA}$的方向用箭头标出，长度（向量的模）为向量的所有分量的平方和的平方根。

![OpenGL_vector](http://ovl0w8qm7.bkt.clouddn.com/OpenGL_vector.jpg)

因为向量只是表示方向和大小的量，因此，向量保持方向不变，在空间内任意平移，平移后的向量都与平移前的向量相等。像上图画的一样，对向量$\vec{BA}$的点B平移到坐标原点得到向量**$\vec{OA'}$**，$\vec{OA'}$ = $\vec{BA}$。

#### 1.2 向量运算

由于时间关系，大家可以认为向量没有除法这回事，所以我们这里会简单地说下向量的加法和乘法。

向量的乘法有两种，一种是点积（也叫点乘），另一种是叉积（也叫叉乘）。点积的计算结果是**标量**，而叉积的计算结果是**矢量**（也就是**向量**）。

如图，点积的计算实际上就是讲两个向量的每个分量分别相乘并求和，也可以通过计算两个向量的模的积再乘以两个向量夹角的余弦值得出。点积的几何意义在于可以计算两个向量之间的夹角。点积满足乘法的交换、分配和结合律。

![OpenGL_vector_dot_mul](http://ovl0w8qm7.bkt.clouddn.com/OpenGL_vector_dot_mul.jpg)

在3D图像空间中，向量叉积会用到单位向量（i, j, k），其计算方法如下：

![OpenGL_vector_x_mul](http://ovl0w8qm7.bkt.clouddn.com/OpenGL_vector_x_mul_1.jpg)

在3D图像空间中，叉积的几何意义很有用，我们可以通过计算向量$\vec{a}$和$\vec{b}$的叉积得到垂直于向量$\vec{a}$和$\vec{b}$的向量$\vec{c}$，$\vec{c}$的模是$\vec{a}$和$\vec{b}$的模的乘积再乘以两个向量之间夹角的正弦值。光看图里的计算结果可能不太清楚计算方式，如果找不到规律的话，请找刘老师详谈。

#### 1.3 右手定则

在物理学中，我们曾经用过“右手定则”：在磁场中，有一根通电导线，将手张开，拇指垂直于其余四根手指，让磁感线垂直于掌心所在的平面，拇指以外的四根手指指向导线电流方向，这样大拇指指向的方向就是导线的运动方向。

上面给出的例子只是“右手定则”在物理学中的一个应用场景，“右手定则”其实是用来判断向量叉积的方向的。

定义矢量$\vec{A}$×矢量$\vec{B}$的结果为一矢量$\vec{C}$，其大小为|$\vec{A}$||$\vec{B}$|sinθ，θ为$\vec{A}$与$\vec{B}$的夹角。其方向满足右手定则：伸直右手四指指向$\vec{A}$的方向，沿着小于π的角度将四指卷向$\vec{B}$的方向，则拇指的指向即为叉积$\vec{C}$的方向。

![右手定则](http://ovl0w8qm7.bkt.clouddn.com/right_hand_principle.jpg)

                                                                                撸牙签示意图

### 2. 矩阵

#### 2.1 何为矩阵？

矩阵，顾名思义，就是矩形的数列，在计算机图形学中，矩阵一般用于给向量做线性变换，不然矩阵也不会出现在我写的这篇文章里是吧。

#### 2.2 矩阵运算

我就不造轮子了，有关矩阵的运算（加、减、数乘、乘法、转置）可以看[这里](https://baike.baidu.com/item/%E7%9F%A9%E9%98%B5/18069?fr=aladdin)。

### 3. 齐次坐标

在“矩阵”小节里，我们知道了点坐标可以写为$[x, y, z]$，那我们怎么去区分点和向量？这时候我们就引入了“齐次坐标”的概念，齐次坐标就是将一个原本是n维的向量用一个n+1维的向量来表示，是指一个用于投影几何里的坐标系统。

《计算机图形学（OpenGL版）》的作者F.S. Hill Jr.曾经说过：“齐次坐标表示是计算机图形学的重要手段之一，它既能够用来明确区分向量和点，同时也更易用于进行仿射（线性）几何变换。”

那么，齐次坐标是如何区分点和向量的呢？非常简单，齐次坐标引入了w分量，若三维空间里的一个点坐标是[x, y, z]的话，那么其齐次坐标就表示为$[x, y, z, 1]$或者$[wx, wy, wz, w] (w \neq 0)$；如果一个向量是$[x, y, z]$的话，那么该向量的齐次坐标表示形式则是$[x, y, z, 0]$。我们随时可以通过截掉最后的$w$分量来获取原本的点/向量，而且我们还可以通过$w$分量来区分点和向量，这是齐次坐标的其中一个好处，另一个好处（更易于进行线性几何变换），会在下边提及。

### 4.三角函数

这个大家肯定都不陌生，给大家复习一下：

$cos(α + β) = cosαcosβ - sinαsinβ \\\\ cos(α - β) = cosαcosβ + sinαsinβ \\\\ sin(α + β) = sinαcosβ + cosαsinβ \\\\ sin(α - β) = sinαcosβ - cosαsinβ$

## 三、线性变换

### 什么是线性变换

线性变换的定义可以看[这里](https://baike.baidu.com/item/%E7%BA%BF%E6%80%A7%E5%8F%98%E6%8D%A2/5904192?fr=aladdin)

假设线性空间V上的一个变换$A$为线性变换，对于V中的任意元素$\vec{a}$，$\vec{b}$和数域$P$中的任意$k$，都有

$A(\vec{a}+\vec{b}) = A(\vec{a}) + A(\vec{b}) \\\\ A(k\vec{a}) = kA(\vec{a})$

### 1.平移

#### 1.1 推导

我们知道，在欧几里德空间（也就是我们常用的直角坐标系、三维空间坐标系等）里，向量的平移可以通过向量加法来实现。像下图的$\vec{OA'}$平移到$\vec{AB}$仅需加一个向量$\vec{OA}$就可以了。

![VectorAddition](http://ovl0w8qm7.bkt.clouddn.com/vector_addition.png)

我们不难看出，向量的平移可以用如下式子表示：

$\begin{bmatrix}x \\\ y \\\ z\end{bmatrix} + \begin{bmatrix}T_x \\\ T_y \\\ T_z\end{bmatrix} = \begin{bmatrix}x + T_x \\\ y + T_y \\\ z + T_z\end{bmatrix}$

其中$T_x$，$T_y$，$T_z$分别是向量在$X$，$Y$，$Z$轴上的平移量。

在线性代数中，一个变换通常是用矩阵x向量的形式来表达，而且GPU对于矩阵乘法的计算非常高效，所以我们总是希望对向量/坐标的变换能用矩阵乘法来实现。

假设存在一个矩阵$A$，$A$乘以一个三维顶点的坐标：

$A\begin{bmatrix}x \\\ y \\\ z\end{bmatrix} = \begin{bmatrix}a_{11} && a_{12} && a_{13} \\\ a_{21} && a_{22} && a_{23} \\\ a_{31} && a_{32} && a_{33} \end{bmatrix} \begin{bmatrix}x \\\ y \\\ z\end{bmatrix} = \begin{bmatrix}a_{11}x && a_{12}y && a_{13}z \\\ a_{21}x && a_{22}y && a_{23}z \\\ a_{31}x && a_{32}y && a_{33}z \end{bmatrix} $

可以看出，无论矩阵$A$里的各个元素取什么值，我们都只能得到$x$，$y$，$z$的线性组合，无法得到一个$x$，$y$，$z$加上一个常数的结果。这时候，齐次坐标的优良特性就凸显出来了。

首先我们先把向量$[x, y, z]$转成齐次坐标的形式$[x, y, z, 1]$，对于这样的坐标，我们很简单就能拼出符合我们预期的矩阵：

$A\begin{bmatrix}x \\\ y \\\ z\end{bmatrix} = \begin{bmatrix}1 & 0 & 0 & T_x \\\ 0 & 1 & 0 & T_y \\\ 0 & 0 & 1 & T_z \\\ 0 & 0 & 0 & 1\end{bmatrix} \begin{bmatrix}x \\\ y \\\ z \\\ 1\end{bmatrix} = \begin{bmatrix}x + T_x \\\ y + T_y \\\ z + T_z \\\ 1\end{bmatrix}$

这样，矩阵$A$就是我们所需要的能实现平移顶点的变换矩阵。

#### 1.2 OpenGL ES计算平移矩阵

在OpenGL ES中，计算平移矩阵的方法很简单，几句代码就搞定：

```java
// 初始化矩阵
float[] matrix = new float[16];
Matrix.setIdentityM(m, 0);
// 计算平移矩阵
Matrix.translateM(m: matrix, offset: 0, x: 3f, y: 2f, z: 1f);
```

这样我们就能得到一个能把顶点在X轴上平移3，在Y轴上平移2，在Z轴上平移1的变换矩阵。

我们不妨看一下里边的实现：

```java
public static void translateM(float[] m, int mOffset, float x, float y, float z) {
        for (int i=0 ; i<4 ; i++) {
            int mi = mOffset + i;
            m[12 + mi] += m[mi] * x + m[4 + mi] * y + m[8 + mi] * z;
        }
}
```

按实现代码来写矩阵的话，我们会得到$\begin{bmatrix}1 & 0 & 0 & 0 \\\ 0 & 1 & 0 & 0 \\\ 0 & 0 & 1 & 0 \\\ T_x & T_y & T_z & 1\end{bmatrix}$，为什么这跟我们推导的矩阵不一样？其实在OpenGL里，矩阵元素排序采用的是以列为主的顺序（column major order），我们不难发现，其实这个矩阵跟我们上面推导的矩阵是互为转置的关系，假设我们推导的矩阵为M，OpenGL的矩阵为A，那么$M = A^T$。

### 2.缩放

#### 2.1 推导

如图，向量$\vec{OP}$在x和y方向上都放大了1.5倍就得到了向量 $\vec{OP_1}$，坐标从(2,1)变换成了(3, 1.5)。这种缩放方式比较符合我们常识中的「放大」或「缩小」，即各个维度上都「放大」或「缩小」相同的倍数。如果在3D空间中的一个对象的各个顶点都「放大」或「缩小」相同的倍数，那么这个3D对象本身就「放大」或「缩小」了相应的倍数。

![向量缩放](http://ovl0w8qm7.bkt.clouddn.com/vector_scale_1.png)

但是，OpenGL ES里的缩放变换可以表达更一般的情形，也就是各个维度上缩放不同的倍数。还是以上图2维向量为例，向量$\vec{OP}$在$x$方向上缩小为原来的0.5倍，在$y$方向上放大为原来的2倍，就得到了向量 $\vec{OP_2}$，坐标从(2, 1)变换成了(1, 2)，这也是一种缩放变换。

从上面的例子可见，缩放变换就是把各个维度的坐标分别「放大」或「缩小」一个倍数。扩展到3D空间，仍然使用4维的齐次坐标，缩放操作用矩阵乘法可以写成：

$\begin{bmatrix}S_x & 0 & 0 & 0 \\\ 0 & S_y & 0 & 0 \\\ 0 & 0 & S_z & 0 \\\ 0 & 0 & 0 & 1\end{bmatrix} \begin{bmatrix}x \\\ y \\\ z \\\ 1\end{bmatrix} = \begin{bmatrix}S_xx\\\ S_yy \\\ S_zz\\\ 1\end{bmatrix}$

缩放比较简单，没啥好说的。

#### 2.2 OpenGL ES计算缩放矩阵

计算缩放矩阵同样灰常简单，初始化矩阵的代码就不放出来了

```java
Matrix.scaleM(m: matrix, offset: 0, x: 1f, y: 1.5f, z: 2f);
```

这样我们就能得到一个能把顶点y坐标放大1.5倍，z坐标放大2倍的缩放矩阵了。我们再来看里边的实现：

```java
public static void scaleM(float[] m, int mOffset, float x, float y, float z) {
        for (int i=0 ; i<4 ; i++) {
            int mi = mOffset + i;
            m[     mi] *= x;
            m[ 4 + mi] *= y;
            m[ 8 + mi] *= z;
        }
}
```

同样这里的矩阵也是跟我们的推导矩阵是互为转置的关系，但是因为缩放矩阵只在对角线上有元素，所以这个矩阵转置前和转置后是一样的。

### 3.旋转

#### 3.1 围绕坐标轴旋转

旋转是相对平移和缩放来说，比较复杂的一种线性变换。在OpenGL ES中，计算旋转矩阵一般是通过一下语句实现：

`Matrix.rotateM(matrix: mMVPMatrix, offset: 0, angle: 30f, x: 0f, y: 0f, z: 1f);`

上面的语句就是计算围绕向量$[0, 0, 1]$旋转30度的变换矩阵，也就是围绕$Z$轴旋转30度。

![PointRotation](http://ovl0w8qm7.bkt.clouddn.com/point_rotation.jpg)
从上面的示意图我们可以看出，点$P(x, y, z)$围绕$Z$轴旋转θ度得到点$P'(x', y', z)$，$z$坐标没有任何变化，我们可以把点$P$和点$P'$投影到$X$-$Y$平面上，得到点$Q$和点$Q'$，$\vec{OQ}$和$\vec{OQ'}$的夹角也等于$θ$。这样，我们就把问题简化成平面几何了：

![PointRotationXYConduct](http://ovl0w8qm7.bkt.clouddn.com/point_rotation_xy_conduct.jpg)

我们通过简单的几何知识可以计算出旋转后的P'坐标与P坐标的关系，计算结果显示，x'和y'都是x和y的线性组合，因此我们可以通过矩阵来表示这个变换：

$M \cdot \begin{bmatrix}x \\\ y \\\ z \\\ 1\end{bmatrix} = \begin{bmatrix}xcosθ + ysinθ \\\ -xsinθ + ycosθ \\\ z \\\ 1\end{bmatrix} $

$M = \begin{bmatrix}cosθ & sinθ & 0 & 0 \\\ -sinθ & cosθ & 0 & 0 \\\ 0 & 0  & 1 & 0 \\\ 0 & 0 & 0 & 1\end{bmatrix}$

矩阵$M$就是我们需要的，将三维空间内任意一个顶点$P$围绕$Z$轴旋转$θ$度得到新顶点$P'$的变换矩阵。

同理，我们不难得出围绕其他坐标轴旋转的变换矩阵：

$M = \begin{bmatrix}cosθ & 0 & sinθ & 0 \\\ 0 & 1 & 0 & 0 \\\ -sinθ & 0 & cosθ & 0 \\\ 0 & 0 & 0 & 1\end{bmatrix}$ 围绕$Y$轴旋转

$M = \begin{bmatrix}1 & 0 & 0 & 0 \\\ 0 & cosθ  & sinθ & 0 \\\ 0 & -sinθ & cosθ & 0 \\\ 0 & 0 & 0 & 1\end{bmatrix}$ 围绕$X$轴旋转

#### 3.2 围绕任意向量旋转

上面我们讲了如何求围绕坐标轴旋转的变换矩阵，但是这种旋转方式属于特殊案例，在实际开发中，我们更多地会围绕某个非坐标轴的向量旋转，这里的计算就比上面的特殊情况复杂得多。

先放结论，在三维空间中任意一顶点/向量$P$围绕任意一个向量$V$旋转$θ$度得到新顶点/向量$P'$的变换矩阵如下：

$M = \begin{bmatrix}x^2(1-cosθ) + cosθ & xy(1-cosθ)-zsinθ & xz(1-cosθ) + ysinθ & 0 \\\ xy(1-cosθ) + zsinθ & y^2(1-cosθ) + cosθ & yz(1-cosθ)-xsinθ & 0 \\\ zx(1-cosθ)-ysinθ & yz(1-cosθ)+xsinθ & z^2(1-cosθ)+cosθ & 0 \\\ 0 & 0 & 0 & 1\end{bmatrix}$

我们不难看出，得到这个变换矩阵的过程会比较繁琐，但是没关系，刘老师会尽可能把推导过程讲诉清楚。

假设我们要对三维空间里的向量$\vec{v}$围绕某个单位向量（向量的模为1）$\vec{r}$旋转$θ$度，可以给出这样的图像：

![VectorRotationPic1](http://ovl0w8qm7.bkt.clouddn.com/vector_rotation_general_pic_1.jpg)

首先，我们把向量$\vec{v}$分解为向量$\vec{v_{||}}$的和向量$\vec{v_⊥}$的和：

$\vec{v} = \vec{v_{||}} + \vec{v_⊥}$                               _式1_

其中$\vec{v_{||}}$是与$\vec{r}$平行的向量，$\vec{v_⊥}$是与$\vec{r}$互相垂直的向量。利用向量点积的几何意义，我们可以得出$\vec{v_{||}}$的表达式：

$\vec{v_{||}} = (\vec{v} \cdot \vec{r}) \cdot \vec{r}$                         _式2_

根据_式1_，$\vec{v_⊥}$可以表现为如下形式：

$\vec{v_⊥} =  \vec{v} - \vec{v_{||}} $

$    = \vec{v}  - (\vec{v} \cdot \vec{r}) \cdot \vec{r}$                        _式3_

旋转变换为T

$T(\vec{v}) = T( \vec{v_{||}} + \vec{v_⊥})$

$= T( \vec{v_{||}}) + T(\vec{v_⊥})$

$= \vec{v_{||}} + T(\vec{v_⊥})$                         _式4_

计算变换矩阵的重点就在于计算$T(\vec{v_⊥})$。

我们焦点放到$\vec{v_⊥}$所在的平面上

![VectorRotationGeneral2](http://ovl0w8qm7.bkt.clouddn.com/vector_rotation_general_pic_2.jpg)

我们作一个辅助向量

$\vec{w} = \vec{r} × \vec{v_{⊥}}$

$= \vec{r} × \vec{v}$

这里先证明下为什么$\vec{r} × \vec{v_{⊥}} = \vec{r} × \vec{v}$

首先根据右手定则，$\vec{r} × \vec{v_{⊥}}$和$\vec{r} × \vec{v}$计算出的向量方向必然一致，接下来我们再证明两个向量的模一样即可。

$|\vec{r} × \vec{v_{⊥}}| = |\vec{r}| \cdot |\vec{v_{⊥}}| \cdot sin90$

$ = |\vec{v_{⊥}}| \cdot sin90$                               因为$\vec{r}$是单位向量，模为1

$= |\vec{v_{⊥}}|$                                             sin90 = 1

$|\vec{r} × \vec{v}| = |\vec{r}| \cdot |\vec{v}| \cdot sin\psi$

![VectorRotationPic1](http://ovl0w8qm7.bkt.clouddn.com/vector_rotation_general_pic_1.jpg)

可以看出，$|\vec{v}| \cdot sin\psi = |\vec{v_{⊥}}|$

所以$|\vec{r} × \vec{v}| = |\vec{r}| \cdot |\vec{v}| \cdot sin\psi = |{\vec{v_{⊥}}}|$

因此

$\vec{w} = \vec{r} × \vec{v_{⊥}} = \vec{r} × \vec{v}$                         _式5_

回到$\vec{v_{⊥}}$所在平面的向量分解示意图：

![VectorRotationGeneral2](http://ovl0w8qm7.bkt.clouddn.com/vector_rotation_general_pic_2.jpg)

我们可以得出

$T(\vec{v_{⊥}}) = cosθ\vec{v_{⊥}} + sinθ\vec{w}$

$= cosθ\vec{v_{⊥}} + sinθ(\vec{r} × \vec{v})$                 _式6_

联立_式2_，_式3_，_式4_，_式5_，_式6_，我们可以展开$T(\vec{v})$：

$T(\vec{v}) = \vec{v_{||}} + T(\vec{v_{⊥}})$

$= (\vec{v} \cdot \vec{r}) \cdot \vec{r} + cosθ\vec{v_{⊥}} + sinθ(\vec{r} × \vec{v})$

$= (\vec{v} \cdot \vec{r}) \cdot \vec{r} + cosθ[\vec{v} - (\vec{v} \cdot \vec{r}) \cdot \vec{r}]+ sinθ(\vec{r} × \vec{v})$

$= (\vec{v} \cdot \vec{r}) \cdot \vec{r} + cosθ\vec{v} - cosθ(\vec{v} \cdot \vec{r}) \cdot \vec{r}+ sinθ(\vec{r} × \vec{v})$

$= (1 - cosθ)(\vec{v} \cdot \vec{r}) \cdot \vec{r} + cosθ\vec{v} + sinθ(\vec{r} × \vec{v})$                   _式7_

看起来好像有点复杂，要求$T(\vec{v})$的话，我们就需要先求出$(\vec{r} × \vec{v})$和$(\vec{v} \cdot \vec{r}) \cdot \vec{r}$。

我们要求解$(\vec{r} × \vec{v})$，不妨先证明一下叉积的矩阵乘法表现形式：

设$\vec{a} = (x_1, y_1, z_1), \vec{b} = (x_2, y_2, z_2)$

单位向量  $\vec{i} = (1, 0, 0,) \\\\ \vec{j} = (0,1,0) \\\\ \vec{k} = (0, 0, 1)$

$\vec{a}×\vec{b}  = \begin{array} {|c c c|}  i & j & k  \\\\ x_1 & y_1 & z_1 \\\\ x_2 & y_2 & z_2 \end{array} = \begin{bmatrix} y_1z_2 - y_2z_1 \\\ -(x_1z_2 - x_2z_1) \\\ x_1y_2 - x_2yz \end{bmatrix} = \begin{bmatrix} y_1z_2 - y_2z_1 \\\  x_2z_1 - x_1z_2  \\\ x_1y_2 - x_2yz \end{bmatrix} $

将上述表达式写成左乘矩阵的形式：

$M \cdot \vec{b} = M \cdot \begin{bmatrix}x_2 \\\ y_2 \\\ z_2\end{bmatrix} = \begin{bmatrix} y_1z_2 - y_2z_1 \\\  x_2z_1 - x_1z_2  \\\ x_1y_2 = x_2yz \end{bmatrix} $

$M = \begin{bmatrix}0 & -z_1 & y_1 \\\ z_1 & 0 & -x_1 \\\ -y_1 & x_1 & 0\end{bmatrix}$

假设$\vec{r} = (x, y, z)$，那么我们有

$\vec{r}×\vec{v} = \begin{bmatrix}0 & -z & y \\\ z & 0 & -x \\\ -y & x & 0\end{bmatrix} \cdot \vec{v}$

再来看$(\vec{v} \cdot \vec{r}) \cdot \vec{r}$，同样，我们尝试把$(\vec{v} \cdot \vec{r}) \cdot \vec{r}$表现为$M \cdot \vec{v}$的形式：

假设$\vec{a} = (x_1, y_1, z_1), \vec{b} = (x_2, y_2, z_2)$

$(\vec{a} \cdot \vec{b}) \cdot \vec{a} = M \cdot {\vec{b}}$，求$M$

$\vec{a} \cdot \vec{b} = x_1x_2 + y_1y_2 + z_1z_2$

$(\vec{a} \cdot \vec{b}) \cdot \vec{a} = (x_1x_2 + y_1y_2 + z_1z_2) \cdot \vec{a} \\ =  (x_1x_2 + y_1y_2 + z_1z_2) \cdot \begin{bmatrix}x_1 \\\ y_1 \\\ z_1\end{bmatrix} \\\\ = \begin{bmatrix}x_1^2x_2 +  x_1y_1y_2 + x_1z_1z_2 \\\ x_1x_2y_1 + y_1^2y_2 + y_1z_1z_2 \\\ x_1x_2z_1 + y_1y_2z_1 + z_1^2z_2\end{bmatrix} \\\\ = M \cdot \begin{bmatrix}x_2 \\\ y_2 \\\ z_2\end{bmatrix} $

$M = \begin{bmatrix} x_1^2 & x_1y_1 & x_1z_1 \\\ x_1y_1 & y_1^2 & y_1z_1 \\\ x_1z_1 & y_1z_1 & z_1^2 \end{bmatrix}$

套到$(\vec{r} \cdot \vec{v}) \cdot \vec{r}$上，就是

$(\vec{v} \cdot \vec{r}) \cdot \vec{r} = (\vec{r} \cdot \vec{v}) \vec{r} \\\\ = \begin{bmatrix}x^2 & xy & xz \\\ xy& y^2 & yz \\\ xz & yz & z^2\end{bmatrix} \cdot \vec{v}$

将这些结果代入_式7_，我们可以得出

$T(\vec{v}) = (1 - cosθ)\begin{bmatrix}x^2 & xy & xz \\\ xy & y^2 & yz \\\ xz & yz & z^2\end{bmatrix} \vec{v} + cosθ\vec{v} + \begin{bmatrix}0 & -z & y \\\ z & 0 & -x \\\ -y & x & 0\end{bmatrix}sinθ\vec{v}$

这里已经很接近我们的最终答案了，但是有点小问题，我们怎么把中间的cosθ融合到矩阵中？这时候就要用到单位矩阵，单位矩阵的性质非常简单粗暴，就是任何左乘单位矩阵的向量，结果都等于向量本身。也就是说，假设单位矩阵为$I$，向量为$\vec{v}$，我们有$I \cdot \vec{v} = \vec{v}$。

我们借助单位矩阵把式子变换一下：

$T(\vec{v}) = (1 - cosθ)\begin{bmatrix}x^2 & xy & xz \\\ xy & y^2 & yz \\\ xz & yz & z^2\end{bmatrix} \vec{v} + cosθ\vec{v} + \begin{bmatrix}0 & -z & y \\\ z & 0 & -x \\\ -y & x & 0\end{bmatrix}sinθ\vec{v} \\\\ = (1 - cosθ)\begin{bmatrix}x^2 & xy & xz \\\ xy & y^2 & yz \\\ xz & yz & z^2\end{bmatrix} \vec{v} + \begin{bmatrix}1 & 0 & 0 \\\ 0 & 1 & 0 \\\ 0 & 0 & 1\end{bmatrix}cosθ\vec{v} + \begin{bmatrix}0 & -z & y \\\ z & 0 & -x \\\ -y & x & 0\end{bmatrix}sinθ\vec{v} \\\\ = \begin{bmatrix}x^2(1-cosθ) + cosθ & xy(1-cosθ)-zsinθ & xz(1-cosθ) + ysinθ \\\ xy(1-cosθ) + zsinθ & y^2(1-cosθ) + cosθ & yz(1-cosθ)-xsinθ\\\ zx(1-cosθ)-ysinθ & yz(1-cosθ)+xsinθ & z^2(1-cosθ)+cos\end{bmatrix} \vec{v}$

我们成功地把$T(\vec{v})$转换为$T(\vec{v}) = M \cdot \vec{v}$的形式。由于我们用的是齐次坐标，所以最后要把矩阵$M$升一维，也就是我们一开始看到的结论：

$M = \begin{bmatrix}x^2(1-cosθ) + cosθ & xy(1-cosθ)-zsinθ & xz(1-cosθ) + ysinθ & 0 \\\ xy(1-cosθ) + zsinθ & y^2(1-cosθ) + cosθ & yz(1-cosθ)-xsinθ & 0 \\\ zx(1-cosθ)-ysinθ & yz(1-cosθ)+xsinθ & z^2(1-cosθ)+cosθ & 0 \\\ 0 & 0 & 0 & 1\end{bmatrix}$

#### 3.3 OpenGL ES计算旋转矩阵

```java
// 围绕z轴旋转30度
Matrix.setRotateM(m: matrix, offset: 0, angle: 30f, x: 0f, y: 0f, z: 1f);
// 围绕向量(1, 3, 5)旋转45度
Matrix.setRotateM(m: matrix, offset: 0, angle: 45f, x: 1f, y: 3f, z: 5f);
```

实现比较长，这里就不贴出来了，算出来的矩阵同样跟我们推导的矩阵是互为转置关系。



## 四、透视投影

因为透视投影是一种非线性变换，所以单独开一个章节来写。

### 1. 什么是透视投影

透视投影是渲染管线(Rendering Pipeline)的重要组成部分，是将相机空间(Camera Space)中的点从视锥体(Frustum)变换到规则观察体(Canonical View Volume)中，待裁剪完毕后进行透视除法的行为。

从几何层面来说，透视投影就是把欧几里德空间里的点映射到一个新的空间的手段。

### 2. 为什么要透视投影

为什么我们需要透视投影？我们可以反过来想这个问题：为什么不用欧几里德三维空间？欧几里德三维空间里每个点都可以找到它们的坐标，而且非常直观，小孩子都能直接利用坐标来作图，为什么不用呢？

原因很简单，欧几里德是一个“理想化”的空间，在这个空间里，平行的线是永远不会相交的，这违背了人类视觉的观感（见下图）。

![Railway](http://ovl0w8qm7.bkt.clouddn.com/railway.jpg)

### 3. 透视投影数学原理

#### 3.1 映射的概念

透视投影在数学层面上，就是坐标点的映射，映射的基本思想很简单：

给定$x\in[a, b]$，找$y\in[c, d]$，使得$\frac{x-a}{b-a} = \frac{y-c}{d-c}$

若$x\notin[a, b]$，则$y\notin[c, d]$。

#### 3.2 透视投影步骤

- 用透视变换矩阵把顶点从视锥体中变换到Clip Space中
- 规则观察体CVV(Canonical View Volume)裁剪后进行透视除法（后边会解释）

![PerspectiveProjectionViewFrustum](http://ovl0w8qm7.bkt.clouddn.com/perspective_projection_view_frustum.png)

#### 3.3 求透视变换矩阵

又到了令人紧张又兴奋的求解部分了，我们慢慢来，先观察X-Z平面：

![PerspectiveProjectionXZ](http://ovl0w8qm7.bkt.clouddn.com/perspective_projection_xz.png)

根据相似三角形的性质，我们可以得出

$\frac{x}{x'} = \frac{z}{z'} = \frac{x}{-N}$

$x' = -N \cdot \frac{x}{z}$

同理，$y' = -N \cdot \frac{y}{z}$

对于任意一点$P = (x, y, z)$我们都可以找到透视投影后的点$P' = (-N \cdot \frac{x}{z}, -N \cdot \frac{y}{z}, -N)$

投影结果中，$z'$始终等于-N，但是在渲染管线中，后续会对Fragment有操作，如执行Z缓冲消隐算法，我们有必要z值保留下来，所以$P' = (-N \cdot \frac{x}{z}, -N \cdot \frac{y}{z}, z)$

为了让程序易于处理，我们如果把$P'$写成如下形式

$P' = (-N \cdot \frac{x}{z}, -N \cdot \frac{y}{z}, -\frac{az + b}{z})$

我们把$P'$转成齐次坐标的形式：

$P' =  (-N \cdot \frac{x}{z}, -N \cdot \frac{y}{z}, -\frac{az + b}{z}, 1) $

再把所有分量乘以$-z$，得到

$P' =  (Nx, Ny, az + b, z)$

这样我们就可以用下面的矩阵来表示这种变换：

$\begin{bmatrix}N & 0 & 0 & 0 \\\ 0 & N & 0 & 0 \\\ 0 & 0 & a & b \\\ 0 & 0 & -1 & 0\end{bmatrix} \cdot \begin{bmatrix}x \\\ y \\\ z \\\ 1\end{bmatrix} = \begin{bmatrix}Nx \\\ Ny \\\ az+b \\\ -z\end{bmatrix}$

再把$\begin{bmatrix}Nx \\\ Ny \\\ az+b \\\ -z\end{bmatrix}$所有分量都除以$-z$

得到$\begin{bmatrix}-N\frac{x}{z} \\\ -N\frac{y}{z} \\\ -\frac{az+b}{z} \\\ 1\end{bmatrix}$

这个过程就叫透视除法，会损失一些必要的信息，如原始的$z$值

为什么要把$z$写成$-\frac{az+b}{z}$？有两点原因：

- $P'$的3个分量统一除以$-z$，易于使用齐次坐标变换普通坐标的方式，后续使用相对统一和高效。
- CVV是一个x, y, z范围都是[-1, 1]的立方体，便于多边形的裁剪。我们可以选择适当的a和b使得$-\frac{az+b}{z}$在$z = -N$时的值等于-1，在$Z = -F$时的值等于1。

联立两个式子

$-\frac{az+b}{z} = -1$, when $z = -N$

$-\frac{az+b}{z} = 1$, when $z = -$F

可得

$a = - \frac{F + N}{F - N}$

$b = - \frac{2FN}{F - N}$

这样，我们就得到了我们的第一版透视变换矩阵:

$\begin{bmatrix}N & 0 & 0 & 0 \\\ 0 & N & 0 & 0 \\\ 0 & 0 & -\frac{F+N}{F-N} & -\frac{2FN}{F-N} \\\ 0 & 0 & -1 & 0\end{bmatrix}$

用第一版投影矩阵可以在z上构建CVV，但是(x, y)都还没限制在[1-, 1]。我们知道$-N \frac{x}{z}$的有效范围是投影屏幕的左边界值（left）和右边界值(right)，即$-N \frac{x}{z} \in [left, right]$；同理，$-N \frac{y}{z} \in [bottom, top]$。

我们想把$-N \frac{x}{z} \in [left, right]$和$-N \frac{y}{z} \in [bottom, top]$映射到[-1, 1]中，即

$\frac{-N\frac{x}{z}-left}{right - left} = \frac{x - (-1)}{1 - (-1)}$

$\frac{-N\frac{y}{z}-bottom}{bottom - top} = \frac{y - (-1)}{1 - (-1)}$

整理一下求x, y的表达式

$x = \frac{2Nx/(-z)}{right - left} - \frac{right+left}{right-left}$

$y = \frac{2Ny/(-z)}{top - bottom} - \frac{top+bottom}{top-bottom}$

代入点$P'$，可得

$P' = \begin{bmatrix}\frac{2Nx/(-z)}{right-left} - \frac{right+left}{right-left} \\\ \frac{2Ny/(-z)}{top-bottom} - \frac{top+bottom}{top-bottom} \\\ -\frac{az+b}{z} \\\ 1\end{bmatrix} \\\\ = \begin{bmatrix}\frac{2Nx}{right-left} + \frac{right+left}{right-left}z \\\ \frac{2Ny}{top-bottom} + \frac{top+bottom}{top-bottom}z \\\ az+b \\\ -z\end{bmatrix}$

老规矩，我们尝试用$M \cdot P = P'$的形式来求变换矩阵

$M \cdot \begin{bmatrix}x \\\ y \\\ z \\\ 1\end{bmatrix} = \begin{bmatrix}\frac{2Nx}{right-left} + \frac{right+left}{right-left}z \\\ \frac{2Ny}{top-bottom} + \frac{top+bottom}{top-bottom}z \\\ az+b \\\ -z\end{bmatrix}$

一波骚操作，我们得到

$\begin{array}{c} M = \begin{bmatrix}\frac{2N}{right - left} & 0 & \frac{right+left}{right-left} & 0 \\\ 0 & \frac{2N}{top-bottom} & \frac{top+bottom}{top-bottom} & 0 \\\ 0 & 0 & a & b \\\ 0 & 0 & -1 & 0\end{bmatrix} \\\\ a = -\frac{F+N}{F-N} \\\\ b = -\frac{2FN}{F - N} \end{array}$

这个矩阵$M$就是我们想要的透视变换矩阵。

最后我们收一下尾，算一下$left$, $right$, $top$和$bottom$的值。

我们可以在$X$-$Z$平面计算视锥体

![PerspectiveProjectionXZFrustum](http://ovl0w8qm7.bkt.clouddn.com/perspective_projection_xz_frustum.jpg)

也可以在$Y$-$Z$平面计算视锥体

![PerspectiveProjectionYZFrustum](http://ovl0w8qm7.bkt.clouddn.com/perspective_projection_yz_frustum.jpg)

一般建议用$Y$-$Z$平面来计算，因为少一个除法，计算效率会高一点。

至此，透视变换矩阵计算完毕。

### 4. OpenGL ES计算透视投影矩阵

```java
// 计算透视矩阵，在y方向上的field of view角度为45度，aspect是宽高比，近平面距离0.1f，远平面距离100f
Matrix.perspectiveM(m: matrix, offset: 0, fovy: 45f, aspect: width/(float) height, zNear: 0.1f, zFar: 100f);
```

可以看出，OpenGL ES是在y方向上计算的，我们先看实现：

```java
public static void perspectiveM(float[] m, int offset, float fovy, float aspect, float zNear, float zFar) {
        float f = 1.0f / (float) Math.tan(fovy * (Math.PI / 360.0));
        float rangeReciprocal = 1.0f / (zNear - zFar);

        m[offset + 0] = f / aspect;
        m[offset + 1] = 0.0f;
        m[offset + 2] = 0.0f;
        m[offset + 3] = 0.0f;

        m[offset + 4] = 0.0f;
        m[offset + 5] = f;
        m[offset + 6] = 0.0f;
        m[offset + 7] = 0.0f;

        m[offset + 8] = 0.0f;
        m[offset + 9] = 0.0f;
        m[offset + 10] = (zFar + zNear) * rangeReciprocal;
        m[offset + 11] = -1.0f;

        m[offset + 12] = 0.0f;
        m[offset + 13] = 0.0f;
        m[offset + 14] = 2.0f * zFar * zNear * rangeReciprocal;
        m[offset + 15] = 0.0f;
    }
```

先按代码把矩阵写出来：

$\begin{bmatrix}\frac{f}{aspect} & 0 & 0 & 0 \\\ 0 & f & 0 & 0 \\\ 0 & 0 & \frac{N+F}{N-F} & -1 \\\ 0 & 0 & \frac{2FN}{N-F} & 0\end{bmatrix}$

$f = 1/tan\frac{fovy}{2} \\\\ N = zNear \\\\ F = zFar $

我们把矩阵转置一下：

$\begin{bmatrix}\frac{f}{aspect} & 0 & 0 & 0 \\\ 0 & f & 0 & 0 \\\ 0 & 0 & \frac{N+F}{N-F} & -1 \\\ 0 & 0 & \frac{2FN}{N-F} & 0\end{bmatrix}^T =\begin{bmatrix}\frac{f}{aspect} & 0 & 0 & 0 \\\ 0 & f & 0 & 0 \\\ 0 & 0 & \frac{N+F}{N-F} & \frac{2FN}{N-F} \\\ 0 & 0 & -1 & 0\end{bmatrix} $

看起来好像跟我们推导的

$M = \begin{bmatrix}\frac{2N}{right - left} & 0 & \frac{right+left}{right-left} & 0 \\\ 0 & \frac{2N}{top-bottom} & \frac{top+bottom}{top-bottom} & 0 \\\ 0 & 0 & -\frac{N+F}{F-N} & \frac{-2FN}{F-N} \\\ 0 & 0 & -1 & 0\end{bmatrix}$

不太一样，可能这里有同学会质问刘老师：“怎么回事？”，但是没关系，刘老师稳得一匹，让刘老师来装完这最后一个B。

OpenGL给出的透视矩阵跟我们推导的矩阵只在前两行上看上去有区别，我们可以先看$\frac{right+left}{right-left}$和$ \frac{top+bottom}{top-bottom}$，由于视锥体是关于Z轴对称的，其实$right+left$和$top+bottom$的值都是0，这里我们就消掉了两个冗余项；

我们再来看$\frac{2N}{top-bottom}$，同理，这个表达式可以写成$\frac{2N}{2top} = \frac{N}{top} = 1/tan\frac{fovy}{2} = f$。

最后，看$\frac{2N}{right-left} = \frac{N}{right} = \frac{N}{top \cdot aspect} = \frac{f}{aspect}$，至此，证明完毕。



## 五、后语

此处应有掌声。