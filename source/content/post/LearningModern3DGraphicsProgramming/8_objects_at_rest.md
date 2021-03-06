+++
date="2017-06-15T12:00:00+06:00" 
title="8. 其他的对象"
categories=["Translation of books"] 
tags=["opengl", "LearningModern3DGraphicsProgramming"]
+++

**原文**: [https://paroj.github.io/gltut/](https://paroj.github.io/gltut/) 

# 第四章 其他的对象

到目前为止，我们看到的都是平面的对象。比如，单个三角形。这一章，会介绍如何创建一个现实中的对象。

## 不真实的世界

orthographic cube例子会渲染一个正交投影的长方体。长方体的长宽高分别是0.5x0.5x1.5。这个例子中的大部分代码和之前的都比较类似。区别的地方是，我们绘制了12个三角形而不是一个三角形。因为一个长方体有六个面，每个面由两个三角形组成。

顶点一样拥有颜色。但是，形成一个面的6个顶点的颜色通常是一样的。

### 面消隐

一个非常值得注意的改变是，在初始化的时候，我们使用了一些新的函数：

**Example 4.1. Face Culling Initialization**

```
void init()
{
    InitializeProgram();
    InitializeVertexBuffer();
    
    glGenVertexArrays(1, &vao);
    glBindVertexArray(vao);

    glEnable(GL_CULL_FACE);
    glCullFace(GL_BACK);
    glFrontFace(GL_CW);
}
```

代码中的最后三行是新的。

`GL_CULL_FACE`标识用来告诉opengl激活面消隐的功能。到目前位置，我们都没有使用过激活面消隐功能。面消隐对于提高性能而言是一个很有用的特征。打个比方，对于长方体而言，我们从任何一个视角看去，最多只能够看到三个面，那么我们就没有必要对看不到的面进行渲染处理。

在窗口坐标系中，通过标准化设备坐标系变换过来的点能够组成一个三角形。三角形的每个顶点在opengl中以一定的顺序表示。这样使得你可以计数三角形顶点的方式。

不管这个三角形的大小和形状，你可以将三角形分为两类：顺时针和逆时针的。如下图所示。左边的图是顺时针的，右边的图是逆时针的。这个的顺序被称为顶点连接顺序。

![](/image/figure4_1_triangle_winding_order.png)  **Figure4.1 Triangle Winding Order**

opengl中的面选择就是基于这个顺序的。`glFrontFace`定义了那种旋转顺序，`GL_CW`和`GL_CCW`分别表示顺时针和逆时针。函数`glCullFace`定义了让哪个面不显示，后面，前面还是都不显示。对应的tag有`GL_BACK`，`GL_FRONT`，`GL_FRONT_AND_BACK`。在这个例子中，我们将背面的face隐藏起来了。

### 缺少视角

此时得到的图像如下：

![](/image/figure4_2_orthographic_prism.png)   **Figure 4.2 Orthographic prism**

这个显示是有问题的。因为，试着想象一下，我们将一个立方体放到眼睛的正前方，只能看到立方体的的一个面，但当我们将立方体往右上方摆放时，能够看到下方和左边的面。但是，这里我们没有看到对应的面，这是为什么呢？

回想一下渲染中发生了什么。在裁剪坐标系中，后面的顶点刚好在正面顶点的正后放，当变换到屏幕坐标系后，这种相对位置没有发生变化，因此只能够看到正前方的面。

这里一定是我们遗漏了某些操作。此处正是“透视”。

## 透视投影

我们最终形成的图像是在屏幕上的二维像素点。3D渲染管线通过转换矩阵将点从裁剪坐标系转换到屏幕坐标系。一旦点到了窗口坐标系，2D三角行就渲染出来了。

根据渲染管线中的定义，投影是将一个维度转变成另一个维度。我们初始的世界坐标系的维度是三维的，因此，渲染管线定义的投影就是将3D的点转换成2D的点。最终渲染得到的三角形就是2D的。

这里我们感兴趣的是有限投影，将对象投影到更低维度的有限空间中。对于3D到2D的投影，是对面进行投影；对于2D到1D的投影是对线进行投影。

正交投影是一个非常简单的投影。当投射到一个坐标轴对齐的表面，如下所示，

![](/image/figure4_3_2d_to_1d_orthographic_projection.png) **Figure4.3 2D to 1D Orthographic Projection**«»

一个场景正交投影到一个黑色的线上。灰色的盒子表示世界中经过投影后可以看到的部分；在灰色部分之外的区域是不可见的。

当投影到任意的直线上时，此时的数学表达将更加的复杂。但是在创建正交投影的时候，垂直于表面的维度将被消除。正是垂直方向上的投影和一致性构成了正交投影。

通过人眼看到的世界并不是正交投影。如果是的话，你就只能够看到瞳孔大小的世界了。

对于我们眼睛看东西可以用针孔相机模型进行模拟。这个模型表现出来的就是透视投影。透视投影就是通过一个点来看到世界在一个面的投影。2d到1d的透视投影看上去如下图：

![](/image/figure4_4_2d_to_1d_perspective_projection.png)  **Figure 4.4 2D to 1D Perspective Projection**

可见，这个投影是基于某一点放射式的展开的。那个点就是眼睛或是相机所在的位置。

从投影的形状可以看出，我们能够看见透视投影使得更大范围的物体可以投影到表面。而正交投影仅仅可以捕捉到位于表面正前方的东西。

在2d中，透视投影的形状是一个等腰梯形。在3d中，这个形状被称为平截头体，好比一个金字塔截掉了顶部。如下所示：

![](/image/figure4_5_viewing_frustum.png)  **Figure 4.5 Viewing Frustum**

### 投影的数学表达

现在我们已经知道了想要做什么，但是我们又该怎么做呢？

我们将做出如下假设：

- 投影平面是和与xy平面平行的，并且是面向-z方向。
- 眼睛的位置是原点（0,0,0）
- 投影平面的大小是在[-1,1]之间。所有在这个范围之外的点是不被绘制的。

是的，这听上去像是规范化坐标系空间。是的，他们之间的相似性并不是巧合，不过现在先不要过多的深入于相似的原因。

我们知道一些关于投影的结果如何产生的细节。透视投影的本质就是将点沿着点到眼睛的方向进行移动。这个问题其实就是一个简单的几何问题，下面是一个2d到1d的视图投影。

![](/image/figure4_6_2d_to_1d_perspective_projection_diagram.png) **Figure 4.6 2D to 1D Perspective Projection Diagram**

P点通过投影得到R点，$P_z$和$E_z$分别是两个点的z轴值。通过相似三角形可得：

$$\vec{R}=\vec{P}\left(\frac{E_z}{P_z}\right)$$

由于这是基于向量的公式，因此这个公式对于3d情况下也同样适用。因此，对于每个顶点的投影用一个简单的公式就可以搞定了。

### 透视除法

基础的透视投影函数是简单的，而且是相当简单，以至于可以直接实现到图形硬件中。

你也许会注意到，缩放可以表达成除法操作符（乘以一个倒数）。同时，也许你也会回忆起裁剪坐标系和标准化坐标系之间的不同之处就是除了一个W。因此，除了在着色器中进行除法操作，我们还可以简单的设置W值，然后让硬件来进行处理。

从裁剪坐标系到标准化坐标系的转换还有一个特殊的名字，叫做透视除法。这样的命名因为它通常用于透视投影；对于正交投影W值设为1.0。

### 相机透视

在我们实现真正的透视投影之前，我们需要处理一个新的问题。对于正交投影，opengl能够对顶点着色器的输出进行自动处理。但对于透视投影，需要一些额外的步骤。实际上，他从本质上改变了世界。

我们的顶点到目前为止都是直接存储的裁剪坐标系中的点。也就是说，缓存对象中的点和顶点坐标系输出的点都是裁剪坐标系中的位置。

opengl会自己将裁剪坐标系中的坐标转换成规范化u坐标系的坐标。透视投影定义了将点转换到裁剪坐标系的过程，因此这些裁剪坐标系中的点，可以对应到3D世界中的点。这个变换的点的输出是裁剪坐标系中的位置，那么输入呢？

因此我们定义了一个新的空间，叫做相机空间。它并不是opengl中原生的空间，而是用户自己构建的空间。相机空间的确切定义会影响到透视投影的过程，因为透视投影需要把点恰好变换到裁剪坐标系中。因此，通过了解透视投影变换的基本过程有助于定义相机空间。这样可以使得相机空间和裁剪空间的透视形式的区别保持最小，这样也可以简化透视投影的逻辑。

相机空间的范围是从正无穷到负无穷的。如果把自己考虑成相机，那么相机空间的x方向是右手方向，y轴正方向是向上的方向，z轴的方向是向后的方向。在裁剪坐标系中，z轴的方向会变成相机空间中z轴方向的反方向。

我们的透视投影变换是基于该空间的。正如前面提到的，投影平面的x，y轴的范围都在[-1,1]范围内，z是在-1的位置上。投影的方向是从-z方向投射过来。

现在，让我们来做一些简单的假设：投影平面的中心是在相机坐标系(0,0,-1)的位置。因此，figure4.6中的E_z为-1。那么接下来就可以得到点与投影后的点之间的比值为，$P_z/-1$。

让眼睛的位置保持固定不变，那么投影平面的移动可以实现放大和缩小的效果。需要做的事情只有当从相机坐标系转换到裁剪坐标系的时候，将xy值乘以一个常量即可。

为了对相机坐标系转化到裁剪坐标系后的点进行比较，下面以2d的方式给出了点变换后的效果。

![](/image/figure4_7_camera_to_ndc_transformation_in_2d.png)  **Figure 4.7 Camera to NDC Transformation in 2D**

需要注意的是，上述图例中，在变换之后，z轴的方向发生了反转。这是因为相机坐标系和标准化坐标系拥有不同的视线方向。在相机坐标系中，相机看向z的负方向，在标准化坐标系中，相机看向的是z的正方向。

如果你对上图中标准话坐标系中的点进行正交投影，得到的结果是相机坐标系的点的透视投影。从效果上来讲，对象该坐标系的变换，就是从一个空间转变到另一个以正交投影看到的物体像是透视后看到的物体的空间中。

### 在深度方向上的透视

到目前为止，我们已经知道了xy轴在透视变换中的变换方式，但是透视变换中z值表示的意思是什么呢？

知道下一章之前我们会忽略掉z值的含义。即使我们需要一些依赖于它的一些变换；如果在标准化坐标系中的一个顶点的坐标范围超出[-1,1]，那么这个点就是不可见的。因为，z和xy一样都要经历透视分割，我们需要考虑在我们的投影中哪些内容是可见的，哪些内容是不可见的。

W值是基于相机坐标系的Z轴的。我们需要将Z值从相机空间中的$[0,-\infty)$到标准化坐标系中的[-1,1]的范围内。由于我们是想要将一个无穷的范围映射到一个有限的范围，我们需要做一些范围上的限制。视景体在xy方向的范围已经是有限的了，类似的，我们需要增加z上的限制。

在视野方向上能够看到的最远的距离为camera zFar，能看到的最近的距离为camera zNear。这样就得到了z方向上的有限范围。

相机zNear能够有效的确定眼睛和投影面之间的偏移。但是，情况并非总是这样的。即使zNear小于1,这将会使得近z平面在投影平面的后面，你同样能够获得有效的投影。投影方式和那些在投影平面前面的物体类似。从数学上来说，两种方式都是一样的。

如果对投影平面进行平移，那么由于投影平面具有固定的大小（范围为[-1,1]），投影到该平面的点会发生变化。

有很多的实现方式，可以将一个有限的范围映射到另一个有限的范围。一个会使人困扰的问题是透视分割本身。两个有限空间的线性变换是很容易就能够理解的。但是经过透视分割之后依然保持线性关系就变得非常困难了。由于我们会除以相机坐标系中的-z，公式就会变得比你想象的复杂。

在下一章中会给出更好的解释，我们将使用下面比较复杂的公式来计算裁剪坐标系中的z值。

**Equation 4.2 Depth Computation**

$$N = zNear; F = zFar; Z_{clip}=\frac{Z_{camera}(F+N)}{N-F}+\frac{2NF}{N-F}$$

公式中zNear，zFar都是正值，zNear可以非常接近0,但不能是0。

### 采用透视的方式来进行绘制

通过上面的介绍，我们可以总结出将一个点从相机坐标系转换到裁剪坐标系的基本步骤：

1. 视景体调整：将相机坐标系中的XY值乘以一个系数。
2. 深度调整：将相机坐标系中的Z值该变成裁剪坐标系中的值。
3. 透视分割：计算$E_z$为-1时的W值。

剩下的就是需要从理论出发，得到实现，创建一个新的顶点着色器，如下：

**Example 4.2 ManualPerspective Vertex Shader**

```
#version 330

layout(location=0) in vec4 position;
layout(location=1) in vec4 color;

smooth out vec4 theColor;

uniform vec2 offset;
uniform float zNear;
uniform float zFar;
uniform float frustumScale;

void main()
{
    vec4 cameraPos = position + vec4(offset.x, offset.y, 0, 0);
    vec4 clipPos;

    clipPos.xy = cameraPos.xy *  frustmScale;
    clipPos.z = cameraPos.z * (zNear + zFar) / (zNear - zFar);
    clipPos.z += 2*zNear*zFar/(zNear - zFar);

    clipPos.w = -cameraPos.z;

    gl_Position = clipPos;
    theColor = color;
}
```

代码中main函数的第一行设定了偏移量，从视线的中心进行偏移。接下来，对xy进行了缩放；计算z值。 裁剪坐标系中的w值恰好正是相机坐标系中z值的负值。opengl可以自动的将裁剪坐标系转换到标准化坐标系。

对应的项目的初始化函数也要进行更改。

**Example 4.3 Program Initialization**

```
offsetUniform = glGetUniformLocation(theProgram, "offset");

frustumScaleUnif = glGetUniformLocation(theProgram, "frustumScale");
zNearUnif = glGetUniformLocation(theProgram, "zNear");
zFarUnif = glGetUniformLocation(theProgram, "zFar");

glUseProgram(theProgram);
glUniform1f(frustumScaleUnif, 1.0f);
glUniform1f(zNearUnif, 1.0f);
glUniform1f(zFarUnif, 3.0f);
glUseProgram(0)
```

我们只对这几个新增加的uniforms仅设置一次。scale为1,表示没有哦变化，z为从-1到-3。因为相机空间中的z值和裁剪坐标系中的有明显的不同，需要对顶点的值进行变化。现在z值将变成-1.25和-2.25。

这些改变后的结果如下：

![](/image/figure-4-8-perspective-prism.png)  **Figure 4.8 Perpective Prism**

## 矩阵有你

因此，现在我们可以将世界坐标系中的点经过投影变换转换为裁剪坐标系中的点。

首先，让我们来看下相机坐标系到裁剪坐标系。S来表示缩放比，N为zNear，F为zFar，那么我们可以得到四个公式。

**Equation 4.3 Camera to Clip Equations**

![](/image/equation_4_3_camera_to_clip_equations.png)

接下来就可以将上式转变成为矩阵形式如下：

**Equation 4.5 Camera to Clip Matrix Transformation**

![](/image/equation_4_5_camera_to_clip_matrix_transformation.png)

上式中最左边和最后边的列向量分别是clip和camera坐标系中的顶点。中间的矩阵是两个空间的点之间的变换矩阵。在图形相关的工作中，我们通常使用4x4的矩阵，具有四行四列。

接下来用矩阵的形式来实现顶点着色器。

**Example 4.4 MatrixPerspective Vertex Shader**

```
#version 330
layout(location=0) in vec4 position;
layout(location=1) in vec4 color;

smooth out vec4 theColor;

uniform vec2 offset;
uniform mat4 perspectiveMatrix;

void main()
{
    vec4 cameraPos = position + vec4(offset.x, offset.y, 0, 0);
    gl_Position = perspectiveMatrix*cameraPos;
    theColor = color;
}
```

opengl着色器语言被设计用来处理图形操作的，那么自然就会有矩阵作为基本类型。上述代码中的mat4就是4x4的矩阵。glsl语言中的类型都有2到4,三个维度的类型。mat4为4x4矩阵，mat2为2x2矩阵，mat2x4是2列4行的矩阵。

这里需要注意的是在这个着色器中没有自己计算变换矩阵；而是通过uniform变量传递进来，因为对于每个点的变换矩阵都是相同的。

向量和矩阵的乘法直接通过`*`实现的。需要留意的是乘法操作是向量和矩阵的顺序。矩阵是在左边的，向量是在右边。矩阵乘法是不可交换的，也就是说`M*v`与`v*M`的结果是不一样的。通常向量也可以被考虑成`1xN`的矩阵，其中N是向量的大小。当你将向量放在矩阵的左边的时候，GLSL将向量认为是`Nx1`的矩阵，只有这样才可以让乘法变得有意义。在我们的计算中需要将向量放在矩阵的右边。

此时初始化中的代码会有所变化：

**Example 4.5 Program Initialization of Perspective Matrix**

```
offsetUniform = glGetUniformLocation(theProgram, "offset");

perspectiveMatrixUnif = glGetUniformLocation(theProgram, "perspectiveMatirx");

float fFrustrumScale = 1.0f, fzNear = 0.5f, fzFar = 3.0f;

float theMatrix[16];
memset(theMatrix, 0, sizeof(float)*6);

theMatrix[0] = fFrustrumScale;
theMatrix[5] = fFrustrumScale;
theMatirx[10] = (fzFar+fzNear)/(fzNear-fzFar);
theMatrix[14] = (2*fzFar*fzNear)/(fzNear-fzFar);
theMatrix[11] = -1.0f;

glUseProgram(theProgram);
glUniformMatrix4fv(perspectiveMatrixUnif, 1, GL_FALSE, theMatrix);
glUseProgram(0);
```

一个4x4的矩阵有16个值。因此在这里我们创建了一个16大小的浮点数组，theMatrix。接下来的式子使用来计算转换矩阵，以及将转换矩阵传递给顶点着色器。需要留意的是，数组形式的矩阵有两种表现方式，一为列优先，另一种为行优先。列优先意味着对于NxM的矩阵，第一列的N个值，随后跟上第二列的N个值，依次形成数组；行优先则是以行为单位依次排列形成数组。

在本例子中，矩阵是列优先的。glUniformMatrix4fv用来将数组传递给着色器，第二个参数表示矩阵的个数，第三个参数表述传入的矩阵是行优先的还是列优先的。GL_TRUE表示行优先的，GL_FALSE表示列优先的。最后一个参数是矩阵数组的起始位置。

变换后得到的结果而之前的相同。

## 世界的方面

如果你运行上例的程序，然后改变窗口的大小。你会发现结果中的矩形被拉长了，如下图：

![](/image/figure_4_10_bad_aspect_ratio.png)  **Figure 4.10. Bad Aspect Ratio**

这是长宽比出现了问题。到目前位置，我们改变窗口的大小的时候，只通过glViewport来告知opengl新的窗口大小。这个改变了opengl的视口变换，即从标准化坐标系到窗口坐标系的变换。NDC空间中的宽高比是1:1，那么u如果窗口宽高比也是1:1，那么两个空间中的形状是一致的。如果窗口空间中的宽高比不是1:1的时候，形状将会被压瘪或拉伸。

那么对于这样的情况将如何进行处理呢？

一个保持1:1的宽高比的很简单的实现方式是：

```
void reshape(int w, int h)
{
    if (w < h)
        glViewport(0, 0, (GLsizei)w, (GLsizei)w);
    else
        glViewport(0, 0, (GLsizei)h, (GLsizei)h);
}
```

此时，当我们改变窗口大小的时候，视口会一直保持方形。但是，如果窗口不是方形的，在右方或是下方会出现较多的空白区域。这些区域是不会被渲染的。

因此，我们需要一个更好的方法。让我们来回到问题本身，标准空间中的方形，如果视口是一个方形，那么得到的结果是方形的；相反的，如果你的窗口不是方形的，而你想要得到一个方形的结果，那么对象在标准坐标系中必定不是方形的。

那么我们最终的目的是想让相机空间中的物体经过变换后转到窗口坐标系中能够保持显示比例。现在我们需要一个矩形的视景体如下：

![](/image/figure-4-11-widescreen-aspect-ratio-frustum.png)  **Figure 4.11 Widescreen Aspect Ratio Frustum**

我们已经可以对视景体的形状做一些控制了。我们不需要移动原点中的眼睛，只需要对xy方向进行适当的缩放即可。当对xy的缩放相同的时候，得到的就是一个方形的视景体。想到得到不是方形的，只要让xy方向的缩放系数保持不同即可。

人类的视野在宽度上看到的比高度上看到的更多。这也说明了为什么大多数的电影的最小宽高比为16:9。因此，大多数时候只要确定了高度，宽度就可以通过宽高比计算得到。

在reshape函数中，可以直接将透视变换中的xy缩放系数确定，如下：

**Example 4.7 Reshape with Aspect Ratio**

```
void reshape(int w, int h)
{
    perspectiveMatrix[0] = fFrustumScale/(w/(float)h);
    perspectiveMatrix[5] = fFrustumScale;

    glUseProgram(theProgram);
    glUniformMatrix4fv(perspectiveMatrixUnif, 1, GL_FALSE, perspectiveMatrix);
    glUseProgram(0);

    glViewport(0, 0, (GLsizei)w, (GLsizei)h);
}
```

这里，我们将x方向上缩放比除以了宽高比。y方向上的缩放比没有发生变化。这种计算是为了xy经过缩放后，得到的是一个方形。

## 回顾

在这一章中，你会学到如下内容：

- 面剔除会按照窗口空间中的部分点的顺序使得一些三角形不会被渲染。
- 透视变换使得在远处的物体看上去比较小，opengl会自动处理w值，将裁剪坐标系转换为标准坐标系。
- 透视变换可以通过矩阵乘法完成。
- 图像显示的合理的宽高比可以通过对相机坐标系中的xy的缩放实现。
