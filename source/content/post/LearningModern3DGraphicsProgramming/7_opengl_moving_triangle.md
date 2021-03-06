+++
date="2017-06-01T12:00:00+06:00" 
title="7. opengl移动三角形"
categories=["Translation of books"] 
tags=["opengl", "LearningModern3DGraphicsProgramming"]
+++

**原文**: [https://paroj.github.io/gltut/](https://paroj.github.io/gltut/) 

# 第三章 opengl移动三角形

这章会讲述如何移动对象。会引入新的着色器相关的技术。

## 移动点

也许能够想到的移动三角形或其他对象最简单的方法是直接改变顶点的数据。从前面的章节中，我们学习了顶点数据是存储在缓存对象中的。于是，更改数据，就是更改缓存区中的数据。cpu_position_offset.cpp就是这么实现的。

整个更改通过了两个过程完成的。首先，针对每个点生成x，y的偏移量,然后将偏移量赋值到每一个点的位置。偏移量的生成`ComputePositionOffset`。

**Example 3.1 Computation of Position Offsets**

```
void ComputePositionOffsets(float &fXOffset, float &fYOffset)
{
    const float fLoopDuration = 5.0f;
    const float fScale = 3.14159f *  2.0f / fLoopDuration;

    float fElapsedTime = glutGet(GLUT_ELAPSED_TIME)/1000.0f;

    float fCurrTimeThroughLoop = fmodf(fElapsedTime, fLoopDuration);

    fXOffset = cosf(fCurrTimeThroughLoop*fScale)*0.5f;
    fYOffset = sinf(fCurrTimeThroughLoop*fscale)*0.5f;
}
```

上述计算得到的偏移量可以产生环状运动，而且在每隔5s，偏移量会回到原来的值(通过fLoopDuration控制)。函数`glutGet(GLUT_ELAPSED_TIME)`用来获取程序开始时，以毫秒为单位的整数时间。`fmodf`用来计算时间的浮点型模。因此，该函数返回值的范围为`[0,fLoopDuration)`。

一旦求得偏移量，偏移量将会被加到顶点坐标中。

**Example 3.2 Adjusting the Vertex Data**

```
void AdjustVertexData(float fXOffset, float fYOffset)
{
    std::vector<float> fNewData(ARRAY_COUNT(vertexdPositions));
    memcpy(&fNewData[0], vertexPositions, sizeof(vertexPositions));

    for (int iVertex=0; iVertex<ARRAY_COUNT(vertexPositions); iVertex+=4)
    {
        fNewData[iVertex] += fXOffset;
        fNewData[iVertex+i] += fYOffset;
    }

    glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
    glBufferSubData(GL_ARRAY_BUFFER,0,sizeof(vertexPositions), &fNewData[0]);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
}
```

这个函数创建了一个临时对象`fNewData`，并将顶点偏移后的值存入其中，然后将带偏移的量通过glBufferSubData将值传递给缓存对象。

`glBufferData`和`glBufferSubData`之间的区别是后一个函数不能够分配内存空间。`glBufferData`可以分配特定大小的内存空间。`glBufferSubData`只用来将数据传递给已经分配了的内存空间中。对一个已经分配了内存空间的缓存对象再次调用`glBufferData`会触发内存空间的重新分配。原先存储的数据将会被丢弃。然而，`glBufferSubData`对一个没有经过`glBufferData`分配空间的缓存对象进行操作会引发错误。

这两个函数可以用c语言中的malloc和memcpy进行对比，其中glBufferSubData就好比memcpy。

`glBufferSubData`也可以用来更新缓存对象中的部分内容。第二个参数是数据即将拷贝到缓存对象空间中的起始地址的偏移量。第三个参数是即将被拷贝的数据的字节数。第四个参数是等待拷贝的数据来源。

**缓存对象使用窍门**。每一次我们需要绘制一些东西的时候，我们需要改变缓存对象中的数据。opengl有一种方式可以实现上述场景，是改变glBufferData的最后一个入参。将`glBufferData(GL_ARRAY_BUFFER, sizeof(vertexPositions), vertexPositions, GL_STATIS_DRAW);`改成`glBufferData(GL_ARRAY_BUFFER, sizeof(vertexPositions), vertexPositions, GL_STREAM_DRAW);`

GL_STATIC_DRAW告诉opengl，你只想要在缓存对象中设置一次数据。GL_DREAM_DRAW告诉opengl你会多次设置数据，通常是一帧一次。当缓存对象的内容需要经常变化时，恰当的使用这些技巧，能够提升缓存对象的性能。在后续的内容中会进一步的给出介绍。

此时渲染函数就是这样的了：

```
void display()
{
    float fXOffset = 0.0f, fYOffset = 0.0f;
    ComputPositionOffsets(fXOffset, fYOffset);
    AdjustVertexData(fXOffset, fYOffset);

    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    glUseProgram(theProgram);

    glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 0, 0);

    glDrawArrays(GL_TRIANGLES, 0, 3);

    glDisableVertexAttribArray(0);
    glUseProgram(0);

    glutSwapBuffers();
    glutPostRedisplay();
}
```

该函数的前三行用来获取偏移量设定新的顶点数据。最后一个函数用来告诉GLUT再次调用display。通常情况下，display函数只在窗口大小发生变化，或是窗口部分被别的窗口覆盖的时候，才会被调用。glutPostRedisplay虽然并不会立刻触发display的调用，但是函数非常迅速的。

如果你运行该例子，会发现，一个绕圈圈的三角形。

## 一个更好的实现方式

对于三个顶点的变换上一章的实现方式还是比较合理的。但如果当数据量达到百万级别的时候。采用上一章的实现方式的话，对对象的移动都会导致百万个顶点的拷贝，以及偏移量的赋值，然后将数值传递给opengl的缓存对象。而且这些还仅仅是在渲染之前的准备工作。这样的话，很可能会使得每一帧之间的间隔会很长。因此，需要一个更好的实现方案。

在显卡的并行能力还不是很显著的时候，上一章的方式几乎是是所有的游戏选择的方案。显卡只能够每帧处理大约10000个三角形。而且，由于大多时候都是先通过cpu对点进行调整，很难处理复杂的图像场景。

GeForce 256是第一块真正意义上能够执行一些顶点操作的显卡。它能够将顶点存储与gpu中，读取它们，并相应的做一些变换，然后将处理完的顶点传输给接下来的管线。GeForce 256中的对顶点的处理非常有用，同时又非常的简单。

基于现代的硬件以及opengl 3.x标准，我们可以通过顶点着色器的方式处理获取更多的便利性。

记住，我们正在做的事情，计算顶点的偏移量。我们可以在顶点着色器中将偏移量赋值给各个顶点。这种方式将更加的简洁。

见下面的positionOffset.vert：

```
#version 330

layout(location=0) in vec4 position;
uniform vec2 offset;

void main()
{
    vec4 totalOffset = vec4(offset.x, offset.y, 0.0, 0.0);
    gl_Position = position + totalOffset;
}
```

上述代码中除了定义了position，还定义了二维向量offset。但是该二维向量是用uniform而不是in，或out进行定义的。这个关键词是有特殊含义的。

**着色器和颗粒性**。着色器每次执行的时候，着色器都会获取变量的新的值，那么这个变量就可以定义成`in`。每一次顶点着色器被调用的时候，都能够从顶点属性缓存中得到不同的输入。这对于顶点位置数组而言是有用的。但对于偏移量而言并不是我们所需要的。我们想要每一个顶点都使用相同的偏移量;那么这个时候你就需要使用”uniform“了。

当一个变量被定义成uniform时，那么这个变量不会与in定义的变量以相同的频率变化。uniform定义的变量只会在render函数调用的时候才会进行赋值变化。在这以后，只有当用户显示的设定新的值之后才会发生变化。

顶点着色器的输入来自顶点属性数组，和缓存对象。而，uniform定义的变量直接由程序对象设定的。

为了这定一个uniform变量，我们需要做两件事。首先是获取uniform变量的位置。类似属性的位置，这里我们使用一个index来表示一个特殊的uniform值。不像属性，我们并不能直接设定这个位置，我们必须通过函数来获取这个位置。在这一章中，通过下面的函数实现的：

`offsetLocation = glGetUniformLocation(theProgram, "offset");`

`glGetUniformLocation`函数获取了以第二个参数命名的uniform变量的位置。需要注意的是，由于uniform变量是定义在着色器中，GLSL并没有给这个变量提供位置。只有当这个uniform变量在程序中被使用了，它才有对应的位置。如果没有对应的位置，那么glGetUniformLocation返回的是-1.在使用该函数之前首先还是需要确认对应的program已经被glUseProgram启用了。此时渲染代码类似如下：

**Example 3.5 Draw with calculated offsets**

```
void display()
{
    float fXOffset = 0.0f, fYOffset = 0.0f;
    ComputePositionOffsets(fXOffset, fYOffset);

    glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    glUseProgram(theProgram);

    glUniform2f(offsetLocation, fXOffset, fYOffset);

    glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 0, 0);

    glDrawArrays(GL_TRIANGLES, 0, 3);

    glDisableVertexAttribArray(0);
    glUseProgram(0);

    glutSwapBuffers();
    glutPostRedisplay();
}
```

## 着色器更多的功能

现在，我们有没有可能将ComputePositionOffsets里面的计算逻辑放到顶点着色器中呢。这个是不行的。因为glutGet(GL_ELAPSED_TIME)不能够在着色器中实现，于是我们需要借助其他的变量。

整个顶点着色器程序如下：

**Example 3.6. Offset Computing Vertex Shader**

```
#version 330

layout(location=0) in vec4 position;
uniform float loopDuration;
uniform float time;

void main()
{
    float timeScale = 3.14159f *  2.0f / loopDuration;
    float currTime = mod(time, loopDuration);
    
    vec4 totalOffset = vec4(cos(currTime*timeScale) * 0.5f,
                            sin(currTime*timeScale) * 0.5f, 0.0f, 0.0f);
    gl_Position = position + totalOffset;
}
```

这个着色器中使用了两个uniforms变量，loopDuration和time。

在这个着色器中，使用了几个标准的GLSL函数，如mod，cos和sin。在上一章中还见过mix。除了这些还有很多其他的标准GLSL函数。

渲染的代码如下：

**Example 3.7. Rendering with time***

``` cpp
void display()
{
    glClearCOlor(0.0f, 0.0f, 0.0f, 0.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    glUseProgram(theProgram);

    glUniform1f(elapsedTimeUniform, glutGet(GLUT_ELAPSED_TIME) / 1000.0f);

    glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 0, 0);

    glDrawArrays(GL_TRIANGLES, 0, 3);

    glDisableVertexAttribArray(0);
    glUseProgram(0);

    glutSwapBuffers();
    glutPostRedisplay();
}
```

你也许会好奇的是，loopDuration是怎么设置的。这个在着色器初始化的时候实现的。

**Example 3.8. Loop Duration Setting**

```
void InitializeProgram()
{
    std::vector<GLuint> shaderList;

    shaderList.push_back(Framework::LoadShader(GL_VERTEX_SHADER, "calcOffset.vert"));
    shaderList.push_back(Framework::LoadShader(GL_FRAGMENT_SHADER, "standard.frag"));

    theProgram = Framework::CreateProgram(shaderList);

    elapsedTimeUniform = glGetUniformLocation(theProgram, "time");

    GLuint loopDurationUnf = glGetUniformLocation(theProgram, "loopDuration");
    glUseProgram(theProgram);
    glUniform1f(loopDurationUnf, 5.0f);
    glUseProgram(0);
}
```

因为loopDurationUnf的变量是不会改变的。因此不需要每次进行设置。

## 多重着色器

能够实现对三角形的移动已经不错了，但是我们还可以在片段着色器中多做一些事情。片段着色器不能够影响对象的位置，但是它能够改变对象的颜色。这就是`fragChangeColor.cpp`中实现的内容。

在`calcColor.frag`中的实现为：

**Example 3.9. Time-based Fragment Shader**

```
#version 330

out vec4 outputColor;

uniform float fragLoopDuration;
uniform float time;

const vec4 firstColor = vec4(1.0f, 1.0f, 1.0f, 1.0f);
const vec4 secondColor = vec4(0.f, 0.f, 0.f, 0.f);

void main()
{
    float currTime = mod(time, fragLoopDuration);
    float currLerp = currTime / fragLoopDuration;

    outputColor = mix(firstColor, secondColor, currLerp);
}
```

该函数和顶点着色器中的实现比较类似。只是这里使用的不是sin，cos函数，而是对两个颜色之间进行了插值。

mix函数实现了两个值之间的插值。当currLerp为0时返回的是第一个参数，当currLerp为1的时候返回的第二个参数。

初始化的代码如下：

**Example 3.10 More Shader Creation**

```
void InitializeProgram()
{
    std::vector<GLuint> shaderList;

    shaderList.push_back(Framework::LoadShader(GL_VERTEX_SHADER, "calcOffset.vert"));
    shaderList.push_back(Framework::LoadShader(GL_FRAGMENT_SHADER, "calcColor.frag"));

    theProgram = Framework::CreateProgram(shaderList);

    elapsedTimeUniform = glGetUniformLocation(theProgram, "time");

    GLuint loopDurationUnf = glGetUniformLocation(theProgram, "loopDuration");
    GLuint fragLoopDurUnf = glGetUniformLocation(theProgram, "fragLoopDuration");


    glUseProgram(theProgram);
    glUniform1f(loopDurationUnf, 5.0f);
    glUniform1f(fragLoopDurUnf, 10.0f);
    glUseProgram(0);
}
```

这里你也许会比较困惑，`time`变量在顶点着色器和片段着色器中都出现了，我们该如何对这个值进行设定呢？GLSL编译模型的一个优点是，顶点着色器和片段着色器被链接成一个对象，具有相同变量名的uniforms会被当成是一个处理。因此，对于`time`变量只有一个位置。

这种处理方式的缺点是，如果你在两个着色器中声明了两个相同名称但不同类型的变量，opengl会给出编译错误的信息，停止产生项目对象。

因此，渲染代码是没有变动的。

**着色器中的全局变量**。 着色器中的全局变量可以用一下修饰符进行定义：`const`，`uniform`，`in`，`out`。const修饰符和c/c++中的类似，表示该变量不能被更改。

## 关于顶点着色器的性能

这些例子都是比较简单的，并且运行的也相当快，但是对于不同操作的性能上的研究还是很重要的。在本章中，我们实现了三种方式移动一个三角形：通过cpu计算偏移量，将对应的点的位置进行更新，然后再将点传递给着色器；将偏移量通过cpu进行计算，在着色器中对点进行更新；将最忌本的参数通过cpu获取，在着色器中进行偏移量计算以及点的更新。那么哪种方式更加合理呢？

这并不是一个简单的问题。但是，几乎所有的情况下，cpu的变换比基于gpu上的变换更慢。这只有当每一帧的变换都是相同的时候才例外。即使是这样，通过在gpu进行一次变换，然后将结果存储在缓存对象中更为合适。这个会在后续的内容中介绍。

另外两种情况，就要依赖于特定的场景了。在我们的例子中。其中一个，我们通过cpu计算得到偏移量，然后将这个值传递给gpu。gpu对每个点进行变换；另一例子中，我们仅仅提供了时间参数，gpu对每个定点都计算得到了相同的偏移量。这意味这gpu做了很多重复性的工作。

尽管如此，这也并不意味这它的执行速度就慢。这取决于数据变换的复杂性。在第二个顶点着色器中，做了sin，cos并不是特别快的运算。这样的话把它交给cpu来计算更加合理些。

当顶点着色器的输入是一种抽象方式的时候，最好在着色器中进行变换运算。这样的时候，我们除了传入顶点的位置之外，还要传入更多常用的信息，然后着色器根据特定的参数来生成顶点最新的位置。同时，在着色器中进行变换计算还能够得到更多的多样性。

## 回顾

在这一章我们学到的内容有：

- 缓冲对象中的内容可以通过`glBufferSubData`进行更新。这类似于c语言中的memcpy
- uniform变量是在glsl语言之外进行设定的。在渲染操作调用的时候才会被改变，在一个渲染操作中这个值是固定的。
- uniform变量对象存储在项目对象中。当项目对喜爱那个显示变换的时候才会变换。
- 在两个不同着色器中定义的相同的uniform对象是同一个对喜爱那个。


