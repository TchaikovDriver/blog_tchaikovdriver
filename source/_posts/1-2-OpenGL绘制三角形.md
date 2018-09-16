---
title: 1.2 OpenGL绘制三角形
date: 2018-09-16 19:46:50
tags: 
  - OpenGL ES
categories: 
  - OpenGL ES
---

1.2 绘制三角形
=======
[TOC]

## 一、扯淡

本文会结合代码来讲述如何绘制三角形，也会尽可能地覆盖疑点，当然一些超纲内容刘老师还没学会，来日方长，日后再说。

绘制一个图形一般需要准备以下东西：

- 画布
- 画笔
- 绘制逻辑

## 二、画布

在Android系统里使用OpenGL绘制，一般会用到`GLSurfaceView`或者`TextureView`，《OpenGL入门》系列如无意外都会采用`GLSurfaceView`作为教材。

`GLSurfaceView`内部维护着GL线程，GL线程启动的时候会创建`EGLContext`以及`EGLSurface`，而`EGLSurface`就是我们需要的画布，虽然我们在绘制的时候并不会直接操作`EGLSurface`，但是没有这个我们什么都画不了。

使用`GLSurfaceView`只需要做简单的配置，配置代码如下：

###1. GLSurfaceView配置代码

```java
 private void initView() {
    // 用OpenGL ES 2.0
    mGLSurfaceView.setEGLContextClientVersion(2);
    // 配置RGBA和深度的size，因为用不上遮罩(mask)，所以stencilSize为0
    mGLSurfaceView.setEGLConfigChooser(8, 8, 8, 8, 16, 0);
    // 配置画笔
    mGLSurfaceView.setRenderer(new SimpleTriangleRenderer());
    // 选择绘制模式为RENDERMODE_CONTINUOUSLY，表示不断刷新和绘制
    // 另一个模式为RENDERMODE_WHEN_DIRTY，仅在调用mGLSurfaceView.requestRender()时才绘制
    mGLSurfaceView.setRenderMode(GLSurfaceView.RENDERMODE_CONTINUOUSLY);
 }

@Override
protected void onStart() {
    super.onStart();
    // 恢复GL线程执行
    mGLSurfaceView.onResume();
}

@Override
protected void onStop() {
    super.onStop();
    // 暂停GL线程
    mGLSurfaceView.onPause();
}
```

至此，`GLSurfaceView`的准备完毕。

## 三、画笔

在上面的示例代码里，我们提到了“画笔”，这个画笔其实就是`GLSurfaceView.Renderer`接口的实现类。绘制一个简单的三角形，画笔并不需要复杂的准备，要做的仅仅时在执行绘制逻辑前把画布清空就可以了。

所以就有了世上最简单的画笔代码：

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
```

至此，世上最简画笔完成了。

## 四、绘制逻辑--三角形

绘制一个三角形需要什么？我们知道，二维图形由点、线、面组成，所以我们首先要确定三个顶点（Vertex）。在OpenGL里，坐标系有x, y, z轴，因此我们的点一般有三个维度。

Vertex的坐标值取值范围是[-1, 1]，数据类型是float。

那就随便先写三个点吧

```java
private static final float[] VERTEX = {
    0f, 1f, 0f, // Surface顶部正中央
    -0.5f, -1f, 0f, // Surface 底部1/4宽度处
    1f, -1f, 0f // Surface底部最右侧
};
```

因为我们画的是2D三角形，所以Z坐标都为0，其实这里也可以不写Z坐标，用二维坐标即可，后面的代码做相应调整就可以了，为了教学方便，这里先用三维坐标。

定义了三个点以后，我们怎么把这三个点在OpenGL的世界里表现出来？

### 1. Vertex Shader

《OpenGL基本概念》里提过的，OpenGL的渲染需要`Vertex Shader`和`Fragment Shader`，顾名思义，存放顶点需要用到`Vertex Shader`，说到Shader我们就要用到GLSL(OpenGL Shading Language)，GLSL以C语言为基础的高阶着色语言，可以在[这里](https://github.com/wshxbqq/GLSL-Card)简单了解下，现在大家先将就着看示例代码吧。

Vertex Shader代码：

```c
// 用于存放顶点的4维向量
attribute vec4 aPosition;
void main() {
  // 读取顶点
  gl_Position = aPosition;
}
```

GLSL程序使用一些特殊的内置变量来获取外部输入，`gl_Position`就是其中一个，用于存放顶点坐标信息，其数据类型为`vec4`，这也是为什么我们明明顶点是三维的，但是`aPosition`却要声明为`vec4`的原因。

### 2. Fragment Shader

《OpenGL基本概念》里还提过光栅化(Rasterization)，也就是把点、线、三角形映射到屏幕上的像素点的过程，这个过程会生成`Fragment`，换句话说，我们要画的三角形就是一个`Fragment`，绘制`Fragment`的话就要准备Fragment Shader：

```c
precision mediump float; // 告诉GPU浮点运算只需要中等精度
void main() {
  gl_FragColor = vec4(1, 0, 0, 1); // Fragment颜色 R G B A
}
```

GLSL在进行光栅化着色的时候，会产生大量的浮点数运算，这些运算可能是当前设备不支持的，所以GLSL提供了3种浮点数精度，我们可以根据不同的设备来使用合适的精度。一般在Fragment Shader最开始的地方加上 `precision mediump float;` 便设定了默认的精度。这样所有没有显式表明精度的变量都会按照设定好的默认精度来处理。

跟`gl_Position`一样，`gl_FragColor`也是GLSL的内置变量，我们通过给该变量赋值从而实现给`Fragment`着色的效果。

### 3. 创建GLSL程序容器

在准备好`Vertex Shader`和`Fragment Shader`后，我们就会想（或者说我会想）怎么使用这两个Shader？这就需要我们创建一个GLSL程序来装载两个Shader，都到这了我就不废话了，请看代码：

```java
public void createProgram(GL10 gl, EGLConfig config) {
  // 创建GLSL程序对象并获取其句柄
  int programHandle = GLES20.glCreateProgram();
  // 加载vertex和fragment的shader
  int vertexShader = loadShader(GLES20.GL_VERTEX_SHADER, VERTEX_SHADER);
  int fragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER, FRAGMENT_SHADER);
  // attach vertex和fragment shader到GLSL程序中
  GLES20.glAttachShader(programHandle, vertexShader);
  GLES20.glAttachShader(programHandle, fragmentShader);
  // 链接并启用GLSL程序
  GLES20.glLinkProgram(programHandle);
  GLES20.glUseProgram(programHandle);
}

private static int loadShader(int type, String shaderCode) {
  // 创建Shader程序并获取其句柄
  int shader = GLES20.glCreateShader(type);
  // 指定Shader源码并绑定
  GLES20.glShaderSource(shader, shaderCode);
  // 编译Shader程序
  GLES20.glCompileShader(shader);
  return shader;
}
```

`GLES20.glLinkProgram(programHandle);`语句，直译就是“链接”GLSL程序，“链接”可以理解为讲编译后并与programHandle attach的Shaders都转换为可以直接在GPU内对应的处理器中（Vertex Processor和Fragment Processor）运行的可执行文件，完整描述可以看[glLinkProgram Description](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glLinkProgram.xhtml)。

`GLES20.glUseProgram(programHandle);`则是把已经attach了Shader和link成功后的GLSL程序投入使用，调用该语句后，后续对Shader对象的操作都不会影响到已经投入使用的GLSL程序了，这种时候可以手动删除两个Shader对象来释放资源。

### 4. 数据准备

在大费周章准备好Vertex Shader，Fragment Shader和GLSL程序后，我们还欠缺关键的一步，就是把数据（三角形的三个顶点）传给Vertex Shader。

#### 4.1 FloatBuffer

由于OpenGL API限制的原因，我们定义的float数组并不能直接使用，为此，我们要用到Java nio包下的`FloatBuffer`。

```java
private static final float[] VERTEX = {
    0f, 1f, 0f, // Surface顶部正中央
    -0.5f, -1f, 0f, // Surface 底部1/4宽度处
    1f, -1f, 0f // Surface底部最右侧
};
// 每个float是4个bytes
private static final int SIZEOF_FLOAT = 4;

FloatBuffer vertexBuffer = ByteBuffer.allocateDirect(VERTEX.length * SIZEOF_FLOAT)
                .order(ByteOrder.nativeOrder())
                .asFloatBuffer()
                .put(VERTEX);
```

如上面代码所示，我们创建了一个 9x4 = 36个bytes的`FloatBuffer`，字节排序顺序跟当前设备保持一致，内容就是我们定义的`VERTEX`数组。

#### 4.2 传递数据

传递数据的代码就三句，但是内容量不少：

```java
// 从GLSL程序中获取Vertex Shader里的aPosition的句柄
int positionLoc = GLES20.glGetAttribLocation(programHandle, "aPosition");
// 启用aPosition
GLES20.glEnableVertexAttribArray(positionLoc);
// 把mVertexBuffer中存储的VERTEX数组的值赋给Vertex Shader程序中的aPosition
// 因为每个顶点都是3维的，所以size是3，stride的值一般是 顶点数x数据类型占用的byte数，float是4个bytes，所以stride = 12
GLES20.glVertexAttribPointer(positionLoc, 3, GLES20.GL_FLOAT, false, 12, mVertexBuffer);
```

逐句分析

`int positionLoc = GLES20.glGetAttribLocation(programHandle, "aPosition");`

在`GLES20.glUseProgram(programHandle);`后，我们用上述语句获取`Vertex Shader`中定义的`attribute vec4 aPosition`变量的句柄（可以理解为这个变量的native引用）。

随后我们用`GLES20.glEnableVertexAttribArray(positionLoc);`启用了`aPosition`attribute，为什么要做这一步呢？因为OpenGL出于性能考虑，在默认情况下，所有Vertex Shader的attribute变量都是关闭的，这意味着数据在Shader端是不可见的，如果不通过显式地调用`GLES20.glEnableVertexAttribArray(positionLoc);`的话，GLSL程序渲染时就无法获取到我们传入的顶点坐标，也就无法进行绘制操作了。

最后，我们通过`GLES20.glVertexAttribPointer(positionLoc, 3, GLES20.GL_FLOAT, false, 12, vertexBuffer);`把前边创建好的`FloatBuffer vertexBuffer`传给`Vertex Shader`的`attribute vec4 aPosition`变量。在该句中，我们声明了`vertexBuffer`中有每个顶点都是三维的，顶点坐标的数据类型是` GLES20.GL_FLOAT`，不需要标准化数据，每个顶点占用12个bytes（三维坐标，也就是三个float，每个float是4 bytes）。

至此，我们的数据也准备完毕了。

### 5. 绘制

铺垫了那么久，总算到了我们蓄谋已久的绘制环节了，下面就是完整的绘制三角形的代码：

```java
GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);
```

调用`GLES20.glDrawArrays`方法之后，GLSL程序会遍历所有enable的数组，按顺序构造出指定的基本图元并绘制出来。

该方法接受3个参数，第一个是mode，表示我们要绘制的图元类型，这里传值

`GLES20.GL_TRIANGLES`，表示我们要画三角形。后两个参数分别是first和count，

相信这两个参数我不说大家也知道是咋回事。

## 五、示例代码

搞了个OpenGLDemo工程，后续授课的示例代码都会放在里边。本期代码可以在该工程的`TriangleDemoActivity`里顺藤摸瓜地看。



Git工程地址：https://github.com/TchaikovDriver/OpenGLDemo