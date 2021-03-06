+++
date="2017-05-30T12:00:00+06:00" 
title="4. 你好，三角形"
categories=["Translation of books"] 
tags=["opengl", "LearningModern3DGraphicsProgramming"]
+++

# 你好，三角形

传统的入门教程在介绍编程语言的时候，通常从“Hello，World！”的程序开始。这样的程序拥有最简单的能够直接输出“Hello, World!”的代码。这是一种熟悉编译系统以及代码执行的很好的一种方式。

使用opengl来写实际的文本的具有一定难度的。在第一章中，我们采用将三角形绘制到屏幕上还取代文本的输出。

## 本章例子中的代码

~~~
#include <iostream>
#include <string>
#include <fstream>
#include <vector>
#include <algorithm>
#include <sstream>
#include <GL/glew.h>
#include <GL/glut.h>
#include <GL/gl.h>

GLuint positionBufferObject;
GLuint theProgram;
const float vertexPositions[] = {
	0.75f, 0.75f, 0.0f, 1.0f,
	0.75f, -0.75f, 0.0f, 1.0f,
	-0.75f, -0.75f, 0.0f, 1.0f,
};

void InitializeVertexBuffer()
{
    glGenBuffers(1, &positionBufferObject);

    glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertexPositions), vertexPositions, GL_STATIC_DRAW);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
}

GLuint CreateShader(GLenum eShaderType, const std::string &strShaderFilename)
{
    GLuint shader = glCreateShader(eShaderType);
    std::ifstream ifile(strShaderFilename.c_str());
    std::stringstream buffer;
    buffer << ifile.rdbuf();
    std::string contents(buffer.str());
    const char* pContents = contents.c_str();

    glShaderSource(shader, 1, &pContents, NULL);

    glCompileShader(shader);
    GLint status;
    glGetShaderiv(shader, GL_COMPILE_STATUS, &status);
    if (status == GL_FALSE)
    {
        GLint infoLogLength;
        glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &infoLogLength);
        GLchar *strInfoLog = new GLchar[infoLogLength+1];
        glGetShaderInfoLog(shader, infoLogLength, NULL, strInfoLog);
        fprintf(stderr, "Compile failure in %d shader: \n%s\n", eShaderType, strInfoLog);
        delete[] strInfoLog;
    }
    return shader;
}

GLuint CreateProgram(const std::vector<GLuint>& vecShader)
{
    GLuint program = glCreateProgram();
    for (size_t i=0; i<vecShader.size(); ++i)
    {
        glAttachShader(program, vecShader[i]);
    }

    glLinkProgram(program);

    GLint status;
    glGetProgramiv(program, GL_LINK_STATUS, &status);
    if (status == GL_FALSE)
    {
        GLint infoLogLength;
        glGetProgramiv(program, GL_INFO_LOG_LENGTH, &infoLogLength);
        printf("Log length: %d\n", infoLogLength);
        GLchar *strInfoLog = new GLchar[infoLogLength+1];
        glGetProgramInfoLog(program, infoLogLength, NULL, strInfoLog);
        fprintf(stderr, "Linker failure: %s\n", strInfoLog);
        delete[] strInfoLog;
    }

    for (size_t i=0; i<vecShader.size(); i++)
    {
        glDetachShader(program, vecShader[i]);
    }
    return program;
}


void InitializeProgram()
{
    std::vector<GLuint> shaderList;
    shaderList.push_back(CreateShader(GL_VERTEX_SHADER, "./triangle.vert"));
    shaderList.push_back(CreateShader(GL_FRAGMENT_SHADER, "./triangle.frag"));

    theProgram = CreateProgram(shaderList);
    std::for_each(shaderList.begin(), shaderList.end(), glDeleteShader);
}

void init()
{
    InitializeProgram();
    InitializeVertexBuffer();
}

void reshape(int w, int h)
{
    glViewport(0, 0, (GLsizei)w, (GLsizei)h);
}

void display()
{
    glClearColor(0.0f, 0.0f, 0.0f,0.0f);
    glClear(GL_COLOR_BUFFER_BIT);

    glUseProgram(theProgram);
    glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0,4,GL_FLOAT,GL_FALSE,0,0);
    glDrawArrays(GL_TRIANGLES, 0, 3);
    glDisableVertexAttribArray(0);
    glUseProgram(0);
    glutSwapBuffers();
}


int main(int argc, char** argv)
{
    glutInit(&argc, argv);

    int width = 500;
    int height = 500;

    glutInitDisplayMode(GLUT_DOUBLE | GLUT_DEPTH);
    glutInitWindowSize(width, height);
    glutInitWindowPosition(300,200);
    int window = glutCreateWindow(argv[0]);

    GLenum err = glewInit();
    if (err != GLEW_OK)
    {
        std::cout << "glewInit failed: " << glewGetErrorString(err) << std::endl;
    }

    init();
    glutDisplayFunc(display);
    glutReshapeFunc(reshape);
    /* glutKeyboardFunc(keyboard); */
    glutMainLoop();
    return 0;
}
~~~

## display函数详解

`display`函数从表面上来看是非常简单的。但是，它的功能是非常复杂的，并且和`init`中的初始化过程有关系。在阅读过程中对于暂时不理解的地方不用纠结，略过继续看即可。

~~~
// Example 1.1. The display Function
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
~~~

前两行用来清空屏幕。`glClearColor`是其中一个状态设置函数，它用来设置清空屏幕的时候使用的颜色。此处将清空颜色设置成了黑色。`glClear`并没有设置opengl的状态，它用来触发屏幕清空操作。其中`GL_COLOR_BUFFER_BIT`参数表示这次清空操作会影响到颜色缓存区，将颜色缓存区中的颜色设置成上述设定的黑色。

接下来的一行代码是用来设置当前的纹理程序，这个纹理程序会被接下去的渲染命令用到。在后面的内容中我们会详细阐述纹理程序是怎么工作的。

再接下去的三行指令都使用来设置opengl上下文中的状态的。这些命令设置了将被渲染的三角形的坐标。他们告诉opengl三角形坐标在内存中的位置。详细内容后续会深入介绍。

`glDrawArrays`从命名上可以发现端倪，这是一个渲染相关的函数。根据当前设定的上下文中的状态，来绘制图形，参数中`GL_TRIANGLES`说明当前绘制的是三角形。

再接下去的两行是简单的清理工作，将渲染完后原先设置的一些参数清空。

最后一行，`glutSwapBuffers`，是`FreeGLUT`的命令，并不是opengl的命令。我们在程序中将opengl的帧缓存设定成了双缓存。这意味着当前显示图像的缓存和当前用于渲染的缓存是两个独立的缓存。因此，我们所有的渲染过程只有在渲染玩之后才会呈现给用户。也就是说，用户不会看到渲染一般的图像。`glutSwapBuffers`是用来将渲染得到的图像呈现给用户。

## 跟着数据走

在**基础简介**的章节中，我们介绍了opengl管线的功能。这次我们将会依据代码再次对管线的内容进行介绍。这里将会更深次的理解到opengl渲染数据相关的细节。

### 顶点传输

光栅化管线的第一个步骤是将顶点转换到裁剪空间。在opengl执行转换之前，必须接受到顶点的列表。因此该管线更初始的步骤是将三角形的顶点数据传输给opengl。

下面是将要被传输的顶点：

~~~
const float vertexPositions[] = {
    0.75f, 0.75f, 0.0f, 1.0f,
    0.75f, -0.75f, 0.0f, 1.0f,
    -0.75f, -0.75f, 0.0f, 1.0f,
}
~~~

每一行都表示一个顶点的四维坐标。这些顶点是四维的原因是裁剪空间中的顶点是四维的。这些顶点已经是裁剪坐标系中的点了。现在我们想要让opengl根据这些点（三角形的三个定点）来渲染三角形。

尽管我们已经有了数据，但是opengl并不能直接使用他们。opengl对于可以读取的内存区域是有一定限制的。你可以自己给顶点分配内存空间，但是opengl并不能直接看到这些内存。因此，我们需要做的第一步事情就是分配一个opengl可见的内存空间，然后将我们的数据填充到里面。这个创建的对象叫做“缓存对象（buffer object）”。

缓存的对象是一个线性的存储空间，受opengl管理和分配。这个内存空间中的内容是受用户控制的，但是用户只有间接的访问能力。可以将这些缓存对象想象成GPU的内存。GPU可以很快的从这个内存中读取内容，因此将数据存储到这里面，是有性能优势的。

本章中的缓存对象是在初始话的时候创建的。如下：

~~~
void InitializeVertexBuffer()
{
    glGenBuffers(1, &positionBufferObject);

    glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertexPositions), vertexPositions, GL_STATIC_DRAW);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
}
~~~

第一行用来创建缓存对象，并将句柄存储到全局变量`positionBufferObject`中。这个对象被创建出来了，但是还没有被分配任何的内存空间。

`glBindBuffer`函数将新创建的缓存对象绑定到`GL_ARRAY_BUFFER`。正如在前面简介中介绍的，opengl中的对象通常必须被绑定到上下文，这样opengl在渲染的时候才会用到这些对象。

`glBufferData`函数进行了两个操作，内存分配和内容拷贝。它为当前绑定到`GL_ARRAY_BUFFER`对象分配内存。通过`sizeof(vertexPositions)`获取分配内存的大小，然后将`vertexPosition`对应大小的内存内容拷贝到`glBufferData`为对象分配的内存中。简单的来讲，可以理解为我们已经拥有了内存中存储的特定大小的数组，然后需要在GPU中分配相应大小的内存空间，最后，将内存拷贝到GPU中的内存中。

`glBufferData`第四个参数的含义将会在后面的内容中给出解释。

第二个绑定操作是起到清理作用。将上下文中`GL_ARRAY_BUFFER`绑定到0,也就是说先前绑定的对象将会被解绑。这个操作从严格意义上来讲并不是必须的。因为，后续的绑定操作，会对当前的对象解绑。但是，进行解绑操作还是一个很好的习惯，除非你对渲染过程有个严格的控制。

通过上述的步骤，我们实现了顶点数据到GPU内存的传输。但是，我们仅仅分配了一定大小的内存，然后用二进制进行填充，opengl并不知道内存中数据的格式。因此，我们需要告诉opengl怎么处理这些存储的数据。

在渲染的代码里我们做了如下操作：

~~~
glBindBuffer(GL_ARRAY_BUFFER, positionBufferObject);
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 0, 0);
~~~

第一个函数我们之前已经说明过了，就是让opengl上下文使用buffer对象。第二个函数，`glEnableVertexAttribArray`将在下一节进行介绍，如果没有这个函数那么第三个函数就不重要了。

第三个函数是最为关键的函数。`glVertexAttribPointer`这个函数里面虽然后Pointer单词，但是它并不是用来处理指针的。而是用来处理缓存对象的。

当渲染发生的时候，opengl会将缓存对象中存储的数据取出来。接着，我们需要告诉opengl的是，这些顶点的数据存储的格式是怎么样的。也就是说，我们需要告诉opengl怎么去识别buffer中存储的数据。

在上面的例子中，我们的数据格式是这样的：

- 我们的位置数据是以32位浮点型数据（c/c++中的float）
- 顶点的每一个位置都有4个数字组成
- 每个顶点数据之间没有空余空间。是在数组中紧密排列的。
- 数组中的第一个值是缓存对象中的起点位置。

`glVertexAttribPointer`将上述四个信息都告诉了opengl。函数的第三个参数是数据中每个值的基本类型，在这个例子中是`GL_FLOAT`，表示32位浮点型。第二个参数表示了有多少个基本类型的数据可以组成一个小的单元，这里是4个。第五个参数用来表示每组数值之间的间隔。在这个例子中，每组数值之间没有间隔，因此这个值是0。第六个参数表示缓存对象的起始的字节偏移量，0表示在缓存对象的起端。

第四个参数会在接下去的章节中介绍。第一个参数会在下面一节中介绍。

这里有一点看上去被忽略的事情是，数据来自于哪个缓存对象。这一点是隐含的。`glVertexAttribPointer`总是绑定到之前绑定到`GL_ARRAY_BUFFER`缓存对象上。因此，函数中没有说明缓存对象的句柄，直接使用了先前绑定的句柄。

这个函数在接下去的章节中会给出详细介绍。

现在opengl知道从哪里获取顶点数据，于是可以使用下面的函数进行渲染。

~~~
glDrawArrays(GL_TRIANGLES, 0, 3);
~~~

这个函数表面上看起来非常简单，但是它干了很多活。第二个和第三个参数分别表示从顶点数据中读取的初始位置以及个数。0,3就表示被处理的顶点为0,1,2。

第一个参数用来告诉opengl，从顶点中读取的三个数值是用来绘制三角形的。

### 顶点处理和着色器

现在我们已经告诉opengl顶点数据有哪些了，渲染管线的下一个步骤是顶点的处理。这是本章中介绍的两个可编程阶段中的一个，这涉及到着色器的使用。

着色器是运行在gpu上的一个程序。在渲染管线中存在几个可能的着色器阶段，并且没一个都有自己的输入和输出。着色器的目的是对输入进行一定的处理，获得想要的输出。

每个着色器都是在一系列的输入下执行的。这里需要强调的是任何阶段的着色器之间是相互独立的。在单独执行着色器的时候不会出现串扰。着色器对于输入和输出是有严格的定义的。着色器在执行完成之后没有对所有的输出赋值的行为在大多数情况下是不合法的。

顶点着色器正如名称所示，是对一系列顶点进行操作的。严格的来讲，每一个顶点着色器的调用都是针对一个顶点的。这些着色器必须输出用户定义的输出和顶点在裁剪坐标系下对应的位置。这个裁剪坐标系的计算逻辑完全由顶点着色器控制。

opengl中的着色器由opengl着色器语言编写的（GLSL）。这个语言看上去很想C语言，但是和C还是有很大区别的。相对于C语言而言，GLSL有很多的限制，如不能有递归的逻辑等。下面我们给出了顶点着色器的示例:

~~~
#version 330

layout(location = 0) in vec4 position;
void main()
{
    gl_Position = position;
}
~~~

上面的代码看上去是非常简单的。第一行表示了这个着色器使用的GLSL的版本是3.30。所有的GLSL着色器都需要声明版本号。

接下去的一行定义了顶点着色器的输入。输入变量的名字为`position`，类型是`vec4`：一个四维的浮点型向量。这一句话也说明了变量的布局位置是0,这个在下面会给出更多的解释。

和C语言类似的是，着色器的执行也都是从`main`函数的调用开始的。这个着色器非常简单，仅仅实现了将输入`position`的值拷贝给`gl_Position`的变量。这个变量没有在着色器中定义；因为它是一个在每个着色器中都事先定义好了的标准变量。如果在GLSL代码中发现一个变量是以`gl_`开始的，那么这个变量一定是内置的变量。在自己对变量生命定义的时候，是不允许使用`gl_`作为开头的。

`gl_Position`的定义类似如下：

~~~
out vec4 gl_Position;
~~~

顶点着色器至少需要做的事情是产生顶点对应的裁剪坐标系下的位置。`gl_Position`正是顶点在裁剪坐标系中的位置。因此，此处输入的顶点已经是裁剪坐标系下的点了，这个着色器就直接将输入的值复制到输出即可。

**顶点属性**。着色器有输入输出。可以将这想象成函数的入参和返回值。如果一个着色器是一个函数，那么它被调用的时候会有一系列的入参，并且期望得到一系列的输出。

着色器的输入和输出都分别有自己的来源和去处。因此，入参`position`必须在一开始附上来自某个地方的数值。那么这些数值从什么地方来的呢？装载这些数值的地方叫做顶点属性。

你也许会发现“vertex attribute”这两个词好像在哪里出现过。不错，就在`glEnableVertexAttribArray`和`glVertexAttribPointer`两个函数中带有这两个词。

这说明了数据是怎么在opengl管线中传输的。当渲染开始的时候，缓存对象中的顶点数据的读取和设定由函数`glVertexAttribPointer`函数完成。这个函数描述了数据从什么地方来的。通过`glVertexAttribPointer`函数将数据和顶点着色器中的变量名字联系在一起的具体实现是比较复杂的。

顶点着色器中的每个输入参数都有一个对应的位置，叫做index属性。正如上文中提到的，入参的定义如下：

~~~
layout(location = 0) in vec4 position;
~~~

布局的位置（lcoation）就是变量position的index属性。该属性值必须大于等于0,它的取值范围是和硬件相关的。

在代码里，当指向一个属性的时候，都是通过指向index属性实现的。函数`glEnableVertexAttribArray`，`glDisableVertexAttribArray`和`glVertexAttribPointer`的第一个参数都是index属性。在着色器代码里，我们将position变量的index属性设置为0。因此在调用`glEnableVertexAttribArray(0)`函数的时候，就是使得`position`的index属性有效。

下面给除了数据如何传输到顶点着色器的示意图：

![](/image/hello_triangle_figure1.1.png)

没有调用`glEnableVertexAttribArray`，仅仅调用`glVertexAttribPointer`到那个index是没有意义的。enable函数并不一定要放在`glVertexAttribPointer`函数调用之前，但是在渲染命令调用前，enable函数必须要被调用。如果这个属性没被enable，那么渲染的时候将不会用到这个属性。

### 光栅化

到目前已经完成了将3个顶点传递给opengl，并通过顶点着色器将其转换成了裁剪坐标系中的3个位置。接下来，顶点的位置通过将xyz三个分量，分别除以w分量得到标准化设备坐标系中点。在我们的例子中，3个顶点的w分量是1,因此得到的位置已经是标准化设备坐标系中的位置了。

在这之后，顶点位置还需要转换成窗口坐标系中。这个转换过程被成为视图变换。在opengl中对应的函数为glviewport。本章中，窗口大小的变化都会触发该函数的执行。因此本章中reshape函数的实现为：

~~~
void reshape(int w, int h)
{
    glViewport(0, 0, (GLsizei)w, (GLsizei)h);
}
~~~

这个告诉opengl窗口的哪个区域是可以用于渲染的。在这个例子中，我们将整个窗口的可视区域作为渲染区域。如果不调用这个函数，窗口大小的变化就不会对渲染产生影响。同时需要注意的是，上述的代码并没有让渲染保持原来的比例。窗口的收缩和延展会同样影响到三角形的收缩和延展。

glviewport函数调用时，对应的原点位于窗口的左下方。一旦位于窗口坐标系中，opengl可以将三角形的三个顶点通过扫描转换的过程，将三角形转换成一系列的片段。为了完成这样的操作，opengl必须确定这些点表述的是什么。

opengl可以将一系列的点以不同的方式表述。让opengl将点理解成三角形的命令是

~~~
glDrawArrays(GL_TRIANGLES, 0, 3);
~~~

枚举`GL_TRIANGLES`告诉opengl一系列输入点中的每三个点用来构建一个三角形。由于我们仅仅传入了3个点，我们只得到了一个三角形。

**Figure 1.2. Data Flow to Rasterizer**
![](/image/data_flow_to_rasterizer.png)

如果我们传入的是6个点，那么我们就能够得到2个三角形。

### 片段处理

片段着色器是用来计算片段的输出颜色。片段着色器的输入包括片段在窗口空间中的xyz坐标位置。输入中也可以包含一些用户定义的数据。

这里我们使用的片段着色器类似如下：

**Example 1.5. Fragment Shader**

~~~
#version 330

out vec4 outputColor;
void main()
{
    outputColor = vec4(1.0f, 1.0f, 1.0f, 1.0f);
}
~~~

第一行与顶点着色器相同，都是用来表示着色器使用的GLSL语言的版本。第二行代码表示的是片段着色器的输出。输出参数的类型是vec4。

main函数中说明了这个片段着色器的输出是一个分量都为1的四维向量。这样创建得到的是一个白色的三角形。第四个维度表示的是透明度的意思。由于所有的片段着色器输出的点都是对应到窗口坐标系中的，因此这里不需要在对其进行赋值等操作。

**Note**

> 在顶点着色器一节中，我们使用了`layout (location=#)`的表达式的目的是用来提供顶点着色器输入和顶点index属性值之间的关联。这个为了将顶点数组和顶点着色器中的输入关联起来。因此，你接下来会对片段着色器的输出和屏幕中的点之间的关联产生一定的疑惑。
> opengl能够自己进行处理。因为，对于片段着色器输出的位置是唯一确定的，如当前图像要被渲染到屏幕。正因为这样，如果你定义一个片段着色器的输出，那么这个输出值会自动的输出到目标位置的图像中。对于不同的片段着色器的结果输出到不同的图像上是可能的，这样会增加一些复杂度，他的实现也会和输入的处理方式类似。但这里不会对此进行更详细的说明。

## 生成着色器

这里我们将会说明，着色器的相关代码如何传递给opengl，并让它识别的。

着色器语言是用类c语言编写的。因此，opengl对着色器语言的处理方式也类似与对c语言的处理方式。在c语言中，单独的c文件被编译成目标文件，然后，一个或多个目标文件通过链接的方式，生成单个项目，或是动态，静态库。opengl的处理方式非常类似。

一个着色器语言首先被编译成着色器目标对象，然后一个或多个着色器目标对象通过链接的方式生成项目对象。

opengl项目对象包含了用于渲染操作的所有着色器。在本章的示例中，我们有顶点着色器和片段着色器；他们两者通过链接的方式共同组成了项目对象。具体实现的方式类似如下:

~~~
void InitializeProgram()
{
    std::vector<GLuint> shaderList;

    shaderList.push_back(CreateShader(GL_VERTEX_SHADER, strVertexShader));
    shaderList.push_back(CreateShader(GL_FRAGMENT_SHADER, strFragmengShader));

    theProgram = CreateProgram(shaderList);

    std::for_each(shaderList.begin(), shaderList.end(), glDeleteShader);
}
~~~

第一个语句创建的对象用于存储将要被链接的着色器目标。接下去的两条语句将两条着色器代码进行了编译。最后通过CreateProgram将两个着色器目标进行链接，生成项目对象。CreateShader代码如下：

~~~
GLuint CreateShader(GLenum eShaderType, const std::string& strShader)
{
    GLuint shader = glCreateShader(eShaderType);
    const char *pStrShader = strShader.c_str();
    glShaderSource(shader, 1, &pStrShader, NULL);

    glCompileShader(shader);

    GLint status;
    glGetShaderiv(shader, GL_COMPILE_STATUS, &status);
    if (status == GL_FALSE)
    {
        GLint infoLogLength;
        glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &infoLogLength);

        GLchar *strInfoLog = new GLchar[infoLogLength+1];
        glGetShaderInfoLog(shader, infoLogLength, NULL, strInfoLog);
        
        const char *strShaderType = NULL;
        switch(eShaderType)
        {
        case GL_VERTEX_SHADER: strShaderType = "vertex"; break;
        case GL_GEOMETRY_SHADER: strShaderType = "geometry"; break;
        case GL_FRAGMENT_SHADER: strShaderType = "fragment"; break;

        }
        fprintf(stderr, "Compile failure in %s shader:\n%s\n", strShaderType, strInfoLog);
        delete[] strInfoLog;
    }
    return shader;
}
~~~

首先告诉opengl，需要创建什么类型的对象，由`glCreateShader`完成。这个函数可以创建特定类型的纹理对象，如顶点或是片段纹理，因此该函数的入参是纹理对象的类型。由于不同版本的着色器语言会有不同的格式规则，以及对应的预先定义的变量和常量，因此在着色器语言的字符串中首先必须要有说明对应的版本。这样编译器才会知道采用怎样的方式对其进行编译。

**Note**

> 着色器和程序对象都是opengl中的对象。但是，他们的创建方式等和其他的opengl对象有显著的区别。比如，对于缓存对象，它的创建是通过类似与“glGenBuffer”的形式创建的。除此之外还有其他很多的不同之处。

上述代码中的下一个阶段是通过`glShaderSource`将等待编译的着色器语句导入到创建的着色器对象中。该函数的第一个参数是着色器对象，第二个参数是有多少个字符串会被载入到着色器对象。将多个字符串载入到一个纹理对象中，类似于c语言中，将头文件合并到c文件中。第三个参数是将要被载入的字符串数组，最后一个参数是字符串长度的数组，这里传入NULL，表示我们传入的字符串都是以null结尾的。通常情况下，最后一个参数传入NULL就可以了，除非你需要在字符串中使用null字符，才需要传入字符串的长度。

一旦这些字符串在对象中时，他们就可以通过glCompileShader对着色器进行编译。

当编译以后，需要查看编译是否成功。这里，我们使用了`glGetShaderiv`来获取GL_COMPILE_STATUS。如果返回的是GL_FALSE，那么编译就是失败了;否则编译就是成功了。

当编译失败的时候，我们会将错误打印出来。它将消息输出到stderr，解释编译失败的原因。这个错误就类比于c编译器报的错误。

当创建的着色器对象都编译成功后，我们将其传入到CreateProgram函数中。

~~~
GLuint CreateProgram(const std::vector<GLuint> &shaderList)
{
    GLuint program = glCreateProgram();

    for (size_t iLoop=0; iLoop<shaderList.size(); ++iLoop)
        glAttachShader(program, shaderList[iLoop]);

    glLinkProgram(program);

    GLint status;
    glGetProgramiv(program, GL_LINK_STATUS, &status);
    if (status == GL_FALSE)
    {
        GLint infoLogLength;
        glGetProgramiv(program, GL_INFO_LOG_LENGTH, &infoLogLength);
        
        GLchar *strInfoLog = new GLchar[infoLogLength+1];
        glGetProgramInfoLog(program, infoLogLength, NULL, strInfoLog);
        fprintf(stderr, "Linker failure: %s\n", strInfoLog);
        delete[] strInfoLog;
    }

    for (size_t iLoop=0; iLoop<shaderList.size(); ++iLoop)
        glDetachShader(program, shaderList[iLoop]);

    return program;
}
~~~

这个函数是相当简单的。首先用`glCreateProgram`创建空的项目对象。这个函数没有入参。项目对象是所有着色器集合而成的。其次，将所有的创建的着色器通过`glAttachShader`对象挂载到项目对象中。在挂载的时候，并不需要指出各个着色器对象的类型。

当所有的着色器对象都挂载到项目对象后，通过`glLinkProgram`将代码进行链接。与先前的处理方式类似。我们必须通过调用`glGetProgramiv`获取对象链接是否成功。如果链接失败，则输出错误日志，否则返回创建了的项目对象。

**Note**

> 在上述着色器对象中，顶点着色器中的输入`position`的属性位置是着色器自动赋值的。其它的属性需要通过`layout(location=#)`的方式赋值。如果，你忘了对属性的index进行赋值，opengl会自动进行赋值。因此，你可能并不知道一个属性的index。如果你需要查询属性的位置，你可以通过`glGetAttribLocation`获取。

当整个项目成功链接之后，你可以通过`glDeatchShader`将挂载的着色器对象从项目对喜爱那个中删除。项目对象的状态和功能并不会收到影响。这样做的作用是告诉opengl这些着色器对象对于项目对象而言没有作用了。

在项目对象链接之后，还需要将其他的着色器对象彻底删除，使用的函数是`glDeleteShader`。

**项目对象的使用**。在opengl开始渲染的时候，需要告诉opengl执行哪个项目对象，这是需要调用`glUseProgram`函数。在本章的例子中，我们在display函数中调用了两次该函数。第一次传入的参数是`theProgram`，第二次入参是0,告诉opengl没有项目对象用于渲染了。

## 清理工作

本章的例子中创建了很多opengl资源。它分配了缓存对象（占用显存资源）。创建了两个着色器对象，和一个项目对象，这些对象都是存储在opengl的内存中。但是，我们到目前为止仅仅删除了着色器对象。

在这个例子中，并不需要进行额外的清理工作。因为当opengl工作的窗口被关闭的时候，这些资源会被opengl释放。

但是，通常意义上而言，在关掉opengl之前，将这些对象进行释放，是一个比较好的习惯。如果，你将对象放在c++对象中，那么在析够函数中可以将这些资源进行释放。

## 回顾

- 缓存对象是opengl分配的现行内存空间，用来存储顶点数值
- glsl语言编写的着色器被编译成着色器对象，然后通过链接产生项目对象，项目对象将在渲染过程中执行。
- `glDrawArrays`函数用来绘制三角形，使用当前绑定到buffer对象中的点进行渲染。

### opengl函数

glClearColor，glClear：设置清空屏幕的颜色，当glClear调用GL_COLOR_BUFFER_BIT的时候，使用设定的颜色进行清屏。

glGenBuffers,glBindBuffer,glBufferData：这些函数用来创建和操作缓存对象。glGenBuffers创建一个或多个缓存对象，`glBindBuffer`将创建的对象绑定到上下文特定的位置，`glBufferData`用来分配内存，并将特定的数据存入到分配的内存中。

glEnableVertexAttribArray,glDisableVertexAttribArray,glVertexAttribPointer：这些函数用来控制顶点属性数组。`glEnableVertexAttribArray`用来激活属性index，`glDisableVertexAttribArray`用来disable激活的属性index，`glVertexAttribPointer`用来定义顶点数据格式，和具体位置，告诉opengl如何解析缓存对想中的定点数据。

glDrawArrays：用来执行渲染，使用当前激活的顶点属性，以及当前的项目对象。这个函数导致一系列的定点按照一定的顺序进行绘制。

glViewPort：这个函数定义了当前的视口变换。定义了窗口的显示区域。

glCreateShader, glShaderSource, glCompileShader, glDeleteShader：这些函数是用来创建纹理项目对象。`glCreateShader`用来创建特定类型的空的纹理对象。`glShaderSource`将特定的纹理语言字符串设定到特定纹理对象;多次调用该函数会将原先设定的纹理内容覆盖。`glCompileShader`用来对刚设定的纹理对象进行编译。`glDeleteShader`将原先的纹理对象删除。

glCreateProgram, glAttachShader, glLinkProgram, glDetachShader：这些函数用来处理项目对象。`glCreateProgram`创建一个空的项目对象。`glAttachShader`用来将纹理对象绑定到项目对象上。多次调用该函数可以将多个纹理对象绑定到项目对象。`glLinkProgram`将所有绑定的纹理对象进行链接操作。`glDetachShader`用来将项目对象中绑定的纹理对象删除。

glUseProgram：这个函数输入的项目对象设为当前使用的项目。所有发生的渲染操作都是对当前设定的项目所绑定的纹理对象进行操作。如果这个函数的入参设为0,就表示当前没有项目被绑定。

glGetAttribLocation：这个函数使用来获取特定名字的属性的index。
