---
date: "2017-05-30T12:00:00+06:00" 
title: "1. 关于这本书"
categories: ["Translation of books"] 
tags: ["opengl", "LearningModern3DGraphicsProgramming"]
---

# 关于这本书

三维图像处理硬件很快成为了必不可少的组件。很多操作系统能够直接使用三维图像硬件，有些甚至要求需要有3D渲染能力的硬件。同时对于日益增加的手机系统，3D图像硬件，也成为了它们的必备特征。

对于大多数刚刚接触图像以及渲染的人，想要深入理解图像硬件是一件有挑战的事情。

## 为什么阅读这本书

现在已经有很多教授图形学相关的实体书。网络上关于图形学的介绍将更多，常见的形式有，wikis，博客，入门类型的，以及论坛。那么这本书提供了哪些其他书籍所没有提供的内容呢？

从编程性上讲，所有先前提到的资源都是指导初学者按照特定的流程学习新的事物。这样的流程如，将特定的参数设置到较旧的图形显卡上，从而实现特定的渲染过程。这样的按照固定函数流实现编程的方式，通常被认为是教授新的图像编程者的最佳方式。

这种考虑被认为是正确的，因为它能够很快让初学者实现某种特定的功能。如，让一张图片看起来更加的真实。固定函数流的实现方式就像是教骑自行车，照着做就行。

这种方法也存在一些弊端。首先，通过这种方式学习得到的东西在遇到必须通过编程性解决的实际问题时，就没法搞定了。通常的编程能力并不能简单的通过流水线的方式学习到，因此学习得到的只是也不能很好的得到转换。

还有一种潜在的问题是，按照固定的函数来进行学习，会产生掌握某种知识的错觉。学习者可能会认为他们已经对他们为什么这么做有了充分的理解，但是他们仅仅是进行了粘贴复制而已。因此，编程也成为了一种类似于魔术的存在：当你将部分代码粘贴到另一部分代码之前，然后所有的事情貌似开始正常工作了。

这个就使得调试成为了初学者的噩梦。因为，他们从来没有真正理解过代码，也就没有能力去诊断这个特殊的问题到底是什么原因引起的。没有这样的能力，那么调试就成为了猜测性的活动，去猜这个问题到底是什么，以及产生问题的原因。

类似的是，你在理解变成系统之前，并不能很好的使用它。面对可编程的图像硬件意味着面对固定函数式的变成材料通常掩饰的问题。因此，通常在开始阶段，学习进阶的会相对而言比较慢，但当你坚持到最后，你真的会知道每一件事情具体是怎么工作的。

另一个问题是，即使你对固定的函数流程有了很好的掌握，它也将会限制你思考如何解决问题的能力。由于固定函数的不可变性，它会将你限制在问题的解决过程中，而不会考虑问题或解决过程的来源。它会鼓励你将纹理想象成图片，把顶点的数据想象成纹理坐标，颜色，或是位置等。从它的本质可以看出，它会限制创造力和问题的解决能力。

最后，甚至在便携系统中，固定的函数式功能通常不能在对应的图像硬件上实现。对于大多数图形硬件，可变成性是当前的基本需求，在未来可编程性将成为不可或缺的一部分。

这本书提供了关于那些内容是高级概念的的入门级指导。从基础教授可编程的图形渲染。

这本书也包括了其他材料通常会忽略“高级概念”。这些概念并不是真正高级，而是很多入门级材料由于固定函数难以实现而将其忽略。

这本书是最早的，最重要的讲述如何成为图形编程者的书。因此，在任何可能的地方，这本书会以有趣的方式鼓励读者探索图形硬件能够干什么。一个好的图形编程者将图形硬件视为一系列满足他们需求的工具，这本书正式鼓励采取这样的想法。

但是，这本书并不是讲述图形api的书籍。尽管它使用了OpenGL，并且通过OpenGL的方式来讲述渲染的概念，但是它并不是一本详细知道使用OpenGL API的书籍。为了说明不同的图形的概念，该书并没有覆盖所有的API，也没有详细的介绍用到的API所能够支持的所有功能。如果，你已经对图形有了一定的了解，并且需要一本书来交你现代OpenGL的编程模式，那么这本书就是你所需要的。这本书虽然会涉及到不同的API，但是它对概念的介绍会比API的介绍更加的详细。

这本书主要是为了教你如何成为一个图形编程者。并不是特定的图形领域，它主要是为了涵盖几乎所有的基本3D渲染操作。所以，你如果想要成为游戏开发，CAD程序设计者，做一些计算机视觉，或者其他类似的事情，这本书就恰好适合你。这并不意味着它将会涵盖所有3G图形相关的内容。通俗的讲，它是为了你在3D图形领域进一步发展打基础用的。

最优化的概念在这本书中并没有深入的展开。主要是因为，这一系列相关的内容是高级的话题。最优化通常也与操作系统，硬件密切相关。它们也有可能和使用到的API相关。最优化会在文章的不同地方有所涉及，但对于一个图形编程初学者而言还是一个很复杂的话题。在本书的附录中，有关于最优化的一些大体的介绍。

## 你想要什么

这是一本给图像编程初学者的书，也可以帮助那些对固定函数流的实现方式比较熟悉的读者进一步的理解可编程渲染的原理。

这本书并不适合编程初学者。

希望读者具有能够读懂C和C++代码的能力。如果C/C++知识的掌握程度仅仅是hello world，那么在进一步学习这本书之前，需要掌握更多的编程知识，直到能够写出更有深度的代码。三维图像渲染根本不是一个入门的编程任务，对于传统的图像学习和现代图像学习也是如此。

这本书中的辅导材料也可以用别的编程语言实现。如果你可以阅读C/C++的代码，对于理解这本书中的代码已经足够了。文中还对辅导材料中出现的代码进行了进一步的解释。

任何大量的关于3D渲染的讨论都会有数学的相关论述。这些数学知识都是3D图形学的基础。想要更好的理解这本书，需要你具有基本的集合和代数能力。当必要的时候，本书会介绍一些更高级的数学知识。但是，你至少需要了解集合和代数知识，并不包括线性代数。

这本书中的源代码采用OpenGL作为图形渲染的API。当开始的时候，你并不需要认识什么是OpenGL，但是需要对代码进行编译和运行，所以你还是需要有个OpenGL的开发环境。

需要特别说明的是，你的图形显卡需要能够支持OpenGL3.3的版本。这意味着任何一款GeForce 8xxx或更好的显卡，或是Radeon HD类型的显卡都可以。这些显卡也被称为Direct3D 10显卡，但是你并不需要window vista或window 7的操作系统来运行OpenGL。你还需要从显卡的官方网站上下载和安装最新的驱动。除了显卡驱动和辅助材料中的代码，你并不需要下载或安装其他的东西。

## 开放的

这本书是为了教授图形编程者基本的图形开发知识。最重要的事情是，你乐意去学习它们。

通常情况下，程序员会通过上网搜索或在书本中查找的方式来寻找解决特殊问题的方法。当他们找到的时候，会将代码拷贝到他们的应用中，然后执行代码来检验这段代码是否可行。这种方式能够很快的找到结果，但是这样并不能真正的理解这段代码。

当第一次阅读这本书的时候，可以对一些特别的问题不进行过多的思索。你可以带着问题进行阅读，但是不建议，带着寻找问题答案的方式来驱动你阅读本书。正确的方式，应该是关注内容本身。当你看完这本书的时候，你可以反过来再回顾一下，看一下你是否对想要解决的问题有了深入的了解。

也许，你会发现你有更好的解决方案。

## 这本书的组织结构

这本书被分解成一些普通的主题。每个主题都包含了几章内容。每一章中的辅导材料都描述了相近的问题。几乎每一个概念都是通过代码片段的方式进行阐述。

每一个辅导材料都是从即将被讨论和证实的概念出发。在每一章的结尾都包含了对本章内容的回顾，以及本章内容中涉及的术语。回顾的小节会对本章中出现的概念进行解释，也会包含一些与代码本身相关的建议，这些会有助于你更深层次的理解这些概念。如果这章节中引入了新的OpenGL函数，或OpenGL纹理函数，在这里也会进行回顾。

这本书是面向图形编程初学者的。图形是一个很大的领域，不存在可以涵盖所有内容的一本书。这本书同样没能涵盖所有的技术内容。有些时候，前面提到过的技术内容会在后面的材料中给出详细的介绍，但是并没有足够的空间将所有相关的内容讲清楚。因此，本书中在介绍一些技术内容的时候，会在接下来的小节中粗略的涉及到相关的更高级的技术。这将有助于读者在图形编程的研究中，认识到哪些内容将会对你有所帮助。

在每一章的结尾，都会对本章中出现的术语进行总结。

