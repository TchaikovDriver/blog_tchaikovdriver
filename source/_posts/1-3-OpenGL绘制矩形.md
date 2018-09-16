---
title: 1.3 OpenGL绘制矩形
date: 2018-09-16 20:27:58
tags: 
  - OpenGL ES
categories: 
  - OpenGL ES
---

1.3 绘制矩形
============
[TOC]

## 一、前请提要
在《1.2 绘制三角形》里，我们学习了如何绘制三角形，也知道了在OpenGL ES里面，可绘制的基本图元只有点、线和三角形。那么，这是否意味着我们不能画三角形以外的图形了呢？相信大家看到这里肯定会骂我傻子都知道肯定不可能只能画三角形。
本文将会围绕“如何绘制矩形”来讲述绘制矩形的几种方式。

P.S: `GLSurfaceView`的配置请看《1.2 绘制三角形》，本文主要讲`Renderer`里用到的`DrawProgram`接口的实现。
```java
public class SimpleRenderer implements GLSurfaceView.Renderer {
    /**
     * 承载绘制逻辑的Program
     */
    private final DrawProgram mProgram;

    public SimpleRenderer(DrawProgram program) {
        mProgram = program;
    }

    @Override
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {
        // 指定了清空画布颜色时使用的颜色：黑色
        GLES20.glClearColor(0f, 0f, 0f, 0f);
        // 创建绘制程序
        mProgram.createProgram(gl, config);
    }

    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {
        // 以像素为单位，指定了视口的左下角（在第一象限内，以（0，0）为原点的）位置。width，height————表示这个视口矩形的宽度和高度，根据窗口的实时变化重绘窗口
        GLES20.glViewport(0, 0, width, height);
        mProgram.onSizeChanged(gl, width, height);
    }

    @Override
    public void onDrawFrame(GL10 gl) {
        // 清空颜色缓冲区，也就是用黑色来填充画布
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
        // 执行绘制逻辑
        mProgram.draw(gl);
    }
}

/**
 * OpenGL绘制程序接口
 * <p>
 * Created by LiuYujie on 2018/1/16.
 */
public interface DrawProgram {
    /**
     * 创建GL程序，这两个参数其实我也不知道有啥用，以后再研究吧
     *
     * @param gl     GL
     * @param config EGLConfig
     */
    void createProgram(GL10 gl, EGLConfig config);

    /**
     * Surface尺寸变化时触发
     *
     * @param gl     GL
     * @param width  新Surface宽度
     * @param height 新Surface高度
     */
    void onSizeChanged(GL10 gl, int width, int height);

    /**
     * 执行绘制操作
     *
     * @param gl GL
     */
    void draw(GL10 gl);
}
```
P.S中的P.S：本文的涉及的代码可以在[这里]https://github.com/TchaikovDriver/OpenGLDemo)获取。

## 二、绘制思路

相信不用我说，大家用膝盖都能想到，在只能画点、线、三角形的前提下， 绘制一个矩形，只需要用两个全等直角三角形拼起来就可以了，实际上也确实如此。

那为什么要特意写一篇文章来说这一点？其实是想借着绘制矩形来介绍OpenGL里的几种`Vertex`组合图形的方式。

### 1. glDrawArrays方法

在《OpenGL入门——三角形的绘制》里，我们在准备好`Vertex Shader`和`Fragment Shader`、传递`Vertex`的`FloatBuffer`给GL程序之后，我们调用了

```java
GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);
```

来绘制三角形。调用`GLES20.glDrawArrays`方法之后，GLSL程序会遍历所有enable的数组，按顺序构造出指定的基本图元（这里是三角形`GLES20.GL_TRIANGLES`）并绘制出来。

### 2. 简单粗暴地绘制

用这种方式绘制矩形的话，我们需要6个点，每个三角形3个点，可以在`DrawTriangleProgram`的基础上修改`VERTEX`数组：

```java
private static final float[] VERTEX = {
  			// 第一个三角形
            -.5f, -.5f, 0f,// bottom left
            -.5f, .5f, 0f,// top left
            .5f, .5f, 0f,// top right
  			// 第二个三角形
  			-.5f, -.5f, 0f,// bottom left
  			.5f, .5f, 0f,// top right
            .5f, -.5f, 0f // bottom right
    };
```

接着是修改`draw(GL10 gl)`的实现：

```java
@Override
public void draw(GL10 gl) {
    // 绘制两个三角形
    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 6); // 6个点
}
```

这样，GL程序就会遍历`mVertexBuffer`里的6个点（示意图如下），前三个点ABC绘制为一个三角形，后三个点ADC绘制为另一个三角形，在视觉上这两个三角形合并为一个矩形，这样我们的矩形就绘制完毕了。

![构成矩形的两个三角形](http://ovl0w8qm7.bkt.clouddn.com/OpenGL_DrawRectangle_Points.png)

### 3. 开源节流——GL_TRIANGLE_STRIP

在上一个简单粗暴的绘制方式里，崇尚节约的同学可能会想，一个矩形只需要4个点，上面这种方式其实有两个点是重复的，有没有办法可以复用中间两个点，减少`VERTEX`数组的长度？

答案是肯定的，不然我就扯不了那么久了。

我们可以看`GLES20.glDrawArrays(int mode, int first, int count)`方法签名，第一个参数`mode`指代的是绘制模式，绘制三角形的模式，只要好奇心足够，我们会发现绘制三角形一共有三种模式：

`GLES20.GL_TRIANGLES`，`GLES20.GL_TRIANGLE_STRIP`和`GLES20.GL_TRIANGLE_FAN`。

第一种模式`GLES20.GL_TRIANGLES`是遍历顶点数组，以3个为一组，3个点组合为一个三角形，依次绘制，假设传入的顶点数为n，则绘制的三角形数量是n/3。

第二种模式`GLES20.GL_TRIANGLE_STRIP`是遍历顶点数组，以**相邻的3个点**为组合，依次绘制，假设传入的定点数为n，则绘制的三角形数量是n-2（n >= 3）。

第三种模式`GLES20.GL_TRIANGLE_FAN`是遍历顶点数组，以**首个顶点**为所有要绘制的三角形的其中一个点，剩下的顶点按顺序，以2个点为单位，每个单位分别与第一个顶点组合成一个三角形。

在这小节里，我们用`GLES20.GL_TRIANGLE_STRIP`来绘制矩形。

首先，我们要确定顶点顺序，这一点很关键，OpenGL绘制图形是按顶点顺序来组合图形的，错误的顶点顺序组合而成的图形可能会跟我们想要的结果有出入。

![构成矩形的两个三角形](http://ovl0w8qm7.bkt.clouddn.com/OpenGL_DrawRectangle_Points.png)

还是这张图，可以看出，在上一小节中我们重复声明的两个点是点A和点C，所以`VERTEX`数组应该这样修改：

```java
private static final float[] VERTEX = {
            -.5f, .5f, 0, // top left B
            .5f, .5f, 0, // top right C
  			-.5f, -.5f, 0,// bottom left A
            .5f, -.5f, 0 // bottom right D
    };
```

根据注释内容可以看出，B，C，A三个点组成矩形上半部分，C，A，D三个点组成矩形的下半部分，最后，我们修改一下绘制代码：

```java
@Override
public void draw(GL10 gl) {
    // 绘制两个三角形
    GLES20.glDrawArrays(GLES20.GL_TRIANGLE_STRIP, 0, 4); // 4个点
}
```

这样，我们就实现了用4个点来绘制矩形。

### 4. 随心所欲地画——glDrawElements

在上面小节里，我们用`GLES20.GL_TRIANGLE_STRIP`的模式绘制了矩形，这个模式要求我们的`VERTEX`数组严格按照一定顺序来保存顶点坐标。设想一下，如果我们以后要绘制多个矩形，甚至是更复杂的图形，而这些图形中又有非常多共用的点，这种情况下怎么办？

用`GLES20.GL_TRIANGLES`模式绘制会创建很多重复的点；用`GLES20.GL_TRIANGLE_STRIP`的话，视绘制的图形的复杂程度，我们未必能排列出一个恰好覆盖到所有图形的顶点顺序。

设计OpenGL的大佬们当然早就预料到这种case，并给出了解决方案，这个解决方案就是使用`GLES20.glDrawElements(int mode,  int count, int type, Buffer indices)`方法来绘制图形。

这个方法对比`GLES20.glDrawArrays(int mode, int first, int count)`，可以说是换了个思路来组合顶点，后者是按线性顺序以及绘制模式来决定组合图形的顶点，而前者则是通过显式声明的顶点顺序来组合图形，接下来会用代码来详细讲解。

首先，我们还是定义好`VERTEX`数组：

```java
private static final float[] VERTEX = {
  			-.5f, -.5f, 0,// bottom left A
            -.5f, .5f, 0, // top left B
            .5f, .5f, 0, // top right C
            .5f, -.5f, 0 // bottom right D
};
```

可以看出，我们的顶点不能按`GLES20.GL_TRIANGLES_STRIP`的方式来绘制，因为复用的AC两点不在数组的中间。但是，我们知道，绘制这个矩形需要A，B，C和A，C，D来分别组成两个三角形。因此，我们确立了顶点的组合方式分别是第0，1，2和0，2，3个顶点，所以就有了下面的`DRAW_INDEX`数组：

```java
private static final short[] DRAW_INDEX = {
            0, 1, 2, // 第一个三角形，用到左下，左上和右上三个点
            0, 2, 3 // 第二个点，用到左下，右上和右下三个点
};
```

因为`GLES20.glDrawElements`方法接受的index数据结构是Buffer，所以我们要创建一个`ShortBuffer`。

```java
ShortBuffer indices = ByteBuffer.allocateDirect(DRAW_INDEX.length * SIZEOF_SHORT)
                .order(ByteOrder.nativeOrder())
                .asShortBuffer()
                .put(DRAW_INDEX);
indices.position(0);
```

最后，实现绘制方法：

```java
@Override
public void draw(GL10 gl) {
  // 声明index长度，数据类型为unsigned short，以及传入indexBuffer
   GLES20.glDrawElements(GLES20.GL_TRIANGLES, DRAW_INDEX.length, GLES20.GL_UNSIGNED_SHORT,                indices);
}
```

这样，GL程序在绘制时就会按我们指定的顶点顺序，取第0，1，2个顶点画第一个三角形，取第0，2，3个顶点画第二个三角形，两个三角形组合成一个矩形。

在绘制复杂图形时，很多时候使用`GLES20.glDrawElements`方法可以**节省不少顶点数组的空间**，特别是每个顶点包含的信息越多的情况下（如顶点颜色）就更是如此。

### 5. 扩展——GL_TRIANGLE_FAN

#### 5.1 矩形

前面我们有提到`GLES20.GL_TRIANGLE_FAN`绘制模式，这种模式是以**首个顶点**为所有要绘制的三角形的其中一个点，剩下的顶点按顺序，以2个点为单位，每个单位分别与第一个顶点组合成一个三角形。

下面会用这种方式来绘制矩形，需要注意的是，这种方式只会复用第一个顶点，而绘制一个矩形需要复用两个顶点，所以我们不可避免地需要多声明一个顶点：

```java
private static final float[] VERTEX = {
  			-.5f, -.5f, 0,// bottom left A
            -.5f, .5f, 0, // top left B
            .5f, .5f, 0, // top right C
  			.5f, .5f, 0, // top right C
            .5f, -.5f, 0 // bottom right D
};
```

从上面代码可以看出，我们多声明了一个顶点C，所以我们的顶点序列是ABCCD，在`GLES20.GL_TRIANGLE_FAN`模式下，会把A拿出来，剩下的点BCCD按序分为两组，第一组是BC，第二组是CD，这两组分别与顶点A组合，从而得到三角形ABC和三角形ACD，这样就能画出我们想要的矩形了。

实现绘制代码：

```java
GLES20.glDrawArrays(GLES20.GL_TRIANGLE_FAN, 0, VERTEX.length / 3);
```

#### 5.2 拟合圆形

看到这里，可能大家都厌倦了画矩形了，之前我们有提过可以用三角形来拟合复杂图形，现在刚好可以用`GLES20.GL_TRIANGLE_FAN`绘制模式来演示一下如何用三角形来拟合圆形。

![拟合圆形](http://ovl0w8qm7.bkt.clouddn.com/OpenGL_fitting_circle.jpg)

如上图所示，在一个圆里，以圆心为坐标原点建立直角坐标系，把圆形分割为若干等分的扇形，把圆心和扇形的射线与圆边的交点连接起来，就可以得到若干个扇形的内接三角形。当我们把圆形分割得足够多份的时候，我们用三角形拟合出来的图形就跟圆形越接近。

经过简单的数学证明（这里就不赘述了，要证明请找刘老师），我们可以用以下代码来生成顶点坐标：

```java
private static final float[] VERTEX;
static {
        List<Float> vertices = new ArrayList<>();
        // 首个顶点为圆心，即坐标原点(0,0,0)
        vertices.add(0f);
        vertices.add(0f);
        vertices.add(0f);
        // 顶点坐标范围是[-1, 1]
        final float radius = 0.25f;
        // 将圆切割为40等份
        final float deltaDegree = 360f / 20f;
        final float endDegree = deltaDegree + 360f;
        double radian;
        for (float i = 0; i < endDegree; i += deltaDegree) {
            radian = Math.toRadians(i);
            vertices.add((float)(radius * Math.sin(radian)));
            vertices.add((float)(radius * Math.cos(radian)));
            vertices.add(0f);
        }
        VERTEX = new float[vertices.size()];
        for (int i = vertices.size() - 1; i >= 0; i--) {
            VERTEX[i] = vertices.get(i);
        }
    }
```

由于视觉需要，我们会用到透视矩阵变换，这是超纲内容，会在下一期提到，现在大家暂且看看就好。

```java
private static final String VERTEX_SHADER =
                    "attribute vec4 aPosition;\n" +
                    "uniform mat4 uMVPMatrix;\n" +
                    "void main() {\n" +
                    "gl_Position = uMVPMatrix * aPosition;\n" +
                    "}";
```

我们的`VERTEX_SHADER`新增了一个`uMVPMatrix`变量，而且会跟`aPosition`相乘后再赋值给`gl_Position`。这里其实就是对顶点坐标进行了一个矩阵变换。

接着我们要给矩阵赋值，使得这个变换有意义：

```java
// 获取uMVPMatrix的句柄
int mMVPMatrixLoc = GLES20.glGetUniformLocation(programHandle, "uMVPMatrix");
float[] mMVPMatrix = new float[16]; // MVPMatrix是4x4的矩阵
// 给透视矩阵赋值
Matrix.perspectiveM(matrix: mMVPMatrix, offset: 0, fovy: 45f, aspect: width / (float) height, zNear: 0.1f, zFar: 100f);
// 因为设置透视矩阵后Z轴会倒置，要把图形平移到Z轴 -100f~-0.1f的范围内才可以看到
Matrix.translateM(matrix: mMVPMatrix, offset: 0, x: 0, y: 0, z: -3f);
```

最后的绘制逻辑：

```java
@Override
public void draw(GL10 gl) {
    // 把计算好的矩阵传给Vertex Shader中的uMVPMatrix变量
    GLES20.glUniformMatrix4fv(mMVPMatrixLoc, 1, false, mMVPMatrix, 0);
    GLES20.glDrawArrays(GLES20.GL_TRIANGLE_FAN, 0, VERTEX.length / 3);
}
```

运行结果预览：

![拟合圆形预览](http://ovl0w8qm7.bkt.clouddn.com/OpenGL_fitting_circle_example.png)

是不是很黄？