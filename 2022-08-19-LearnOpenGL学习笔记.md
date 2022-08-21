---
title: 2022-08-17-LearnOpenGL学习笔记
description: 随手记一些思路
date: 2022-08-17
#updated: 2022-06-27
categories:
- 读书笔记
tags:
- 读书笔记
---



参考网站https://learnopengl-cn.github.io/

原始网站[Learn OpenGL, extensive tutorial resource for learning Modern OpenGL](https://learnopengl.com/)





# Introduction

略

# Getting started

## OpenGL

略

## Creating a window

这节讲装环境，其实直接去[JoeyDeVries/LearnOpenGL: Code repository of all OpenGL chapters from the book and its accompanying website https://learnopengl.com (github.com)](https://github.com/JoeyDeVries/LearnOpenGL)

下载仓库，用配了CMake 的 Visual Studio 或者 Clion 打开就好



GLFW 是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的**最低限度的接口**。它允许用户**创建OpenGL上下文、定义窗口参数以及处理用户输入**，对我们来说这就够了。

GLAD   是用来管理OpenGL的函数指针。简化（// 定义函数原型 // 找到正确的函数并赋值给函数指针 // 现在函数可以被正常调用了 ）此过程

## Hello Window

讲了一些 API

```cpp

glfwInit();  // 初始化 GLFW

glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);  // glfwWindowHint函数配置 GLFW
// 第一个参数代表选项的名称，我们可以从很多以`GLFW_`开头的枚举值中选择；第二个参数接受一个整型，用来设置这个选项的值。该函数的所有的选项以及对应的值都可以在 [GLFW’s window handling](http://www.glfw.org/docs/latest/window.html#window_hints) 这篇文档中找到。

GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);  // 创建并返回一个GLFWwindow窗口对象
glfwMakeContextCurrent(window);  // 通知GLFW将我们窗口的上下文设置为当前线程的主上下文

gladLoadGLLoader((GLADloadproc)glfwGetProcAddress) // 初始化 GLAD，给 GLAD 传入用来加载系统相关的OpenGL函数指针地址的函数
// 有 typedef void* (* GLADloadproc)(const char *name);
// 意思是定义了一种 GLADloadproc 的类型，这个类型为指向某种函数的指针，接收 char* 并返回 void*
// 所以这句的意思是把函数指针强制类型转换，可以用 gladLoadGLLoader(reinterpret_cast<GLADloadproc>(glfwGetProcAddress)) 代替  

glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);  // 注册回调函数，每次窗口大小被调整的时候被调用
// 回调函数的格式 typedef void (* GLFWframebuffersizefun)(GLFWwindow*,int,int);

void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    // make sure the viewport matches the new window dimensions; note that width and 
    // height will be significantly larger than specified on retina displays.
    glViewport(0, 0, width, height);  // 设置窗口的维度
}

// 渲染循环
while(!glfwWindowShouldClose(window))  // 检查一次GLFW是否被要求退出
{
    processInput(window);

    // 渲染
    // ------
    glClearColor(0.2f, 0.3f, 0.3f, 1.0f);  // 设置清空屏幕所用的颜色
    glClear(GL_COLOR_BUFFER_BIT);  // 清空颜色缓冲

    glfwSwapBuffers(window);  // 交换颜色缓冲
    glfwPollEvents();  // 检查有没有触发什么事件（比如键盘输入、鼠标移动等）、更新窗口状态，并调用对应的回调函数
}

glfwTerminate();  // 释放/删除之前的分配的所有资源

void processInput(GLFWwindow *window)
{
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);  // 把WindowShouldClose属性设置为 true的方法关闭GLFW。下一次while循环的条件检测将会失败，程序将会关闭
}

```

## Hello Triangle

- 顶点数组对象：Vertex Array Object，VAO
- 顶点缓冲对象：Vertex Buffer Object，VBO
- 元素缓冲对象：Element Buffer Object，EBO 或 索引缓冲对象 Index Buffer Object，IBO

图形渲染管线 0

![img](2022-08-19-LearnOpenGL学习笔记.assets/pipeline.png)

图元是由顶点组成的。一个顶点，一条线段，一个三角形或者多边形都可以成为图元

### 顶点输入

```cpp
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};  // 指定三个顶点
unsigned int VBO;
glGenBuffers(1, &VBO);  // 生成一个VBO对象
glBindBuffer(GL_ARRAY_BUFFER, VBO);  // 把新创建的缓冲绑定到GL_ARRAY_BUFFER目标上。从这一刻起，我们使用的任何（在GL_ARRAY_BUFFER目标上的）缓冲调用都会用来配置当前绑定的缓冲(VBO)
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  // 把用户定义的数据复制到当前绑定缓冲
/*
    第四个参数指定了我们希望显卡如何管理给定的数据。它有三种形式：
    GL_STATIC_DRAW ：数据不会或几乎不会改变。
    GL_DYNAMIC_DRAW：数据会被改变很多。
    GL_STREAM_DRAW ：数据每次绘制时都会改变。
*/
```

### 顶点着色器

GLSL 源码

```cpp
#version 330 core  // 版本声明，用核心模式
layout (location = 0) in vec3 aPos;  // 声明所有的输入顶点属性(Input Vertex Attribute)

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);  // gl_Position设置的值会成为该顶点着色器的输出
}
```

### 编译着色器

```cpp
const char *vertexShaderSource = "..."  // 先把刚刚的源码写成字符串
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);  // 创建一个顶点着色器
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);  // 把这个着色器源码附加到着色器对象上
glCompileShader(vertexShader);  // 编译它

int  success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);  // 判断是否编译成功
if(!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);  // 获取错误信息
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```

### 片段着色器

源码

```cpp
#version 330 core
out vec4 FragColor;  // 最终输出的颜色

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
} 
```

编译过程，和顶点着色器类似

```cpp
unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);

```

链接

```cpp
unsigned int shaderProgram;
shaderProgram = glCreateProgram();  // 创建一个程序，并返回新创建程序对象的ID引用
glAttachShader(shaderProgram, vertexShader);  // 把之前编译的着色器附加到程序对象上
glAttachShader(shaderProgram, fragmentShader);  // 把之前编译的着色器附加到程序对象上
glLinkProgram(shaderProgram);  // 链接它们
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);  // 判断是否链接成功
if (!success) {
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);  // 获取错误信息
    std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
}
glUseProgram(shaderProgram);  // 激活这个程序对象，之后每个着色器调用和渲染调用都会使用这个程序对象（也就是之前写的着色器)了
glDeleteShader(vertexShader);  // 把着色器对象链接到程序对象以后，删除着色器对象，我们不再需要它们了
glDeleteShader(fragmentShader);

```

### 链接顶点属性

```cpp
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);  // 告诉OpenGL该如何解析顶点数据
/*
第一个参数，顶点着色器程序里 location = 0	
第二个参数，指定顶点属性的大小。顶点属性是一个vec3，它由3个值组成，所以大小是3。
第三个参数，指定数据的类型，这里是GL_FLOAT(GLSL中vec*都是由浮点数值组成的)。
第四个参数，定义我们是否希望数据被标准化(Normalize)。如果我们设置为GL_TRUE，所有数据都会被映射到0（对于有符号型signed数据是-1）到1之间。我们把它设置为GL_FALSE。
第五个参数叫做步长(Stride)，简单说就是从这个属性第二次出现的地方到整个数组0位置之间有多少字节）。
最后一个参数的类型是void*，所以需要我们进行这个奇怪的强制类型转换。它表示位置数据在缓冲中起始位置的偏移量(Offset)。由于位置数据在数组的开头，所以这里是0。我们会在后面详细解释这个参数。
*/
// 每个顶点属性从一个VBO管理的内存中获得它的数据，就是之前绑定的那个
glEnableVertexAttribArray(0);  // 以顶点属性位置值作为参数，启用顶点属性
```

总结一下，在OpenGL中绘制一个物体，代码会像是这样：

```cpp
// 0. 复制顶点数组到缓冲中供OpenGL使用
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 1. 设置顶点属性指针
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
// 2. 当我们渲染一个物体时要使用着色器程序
glUseProgram(shaderProgram);
// 3. 绘制物体
someOpenGLFunctionThatDrawsOurTriangle();
```

如果顶点和物体太多，绑定正确的缓冲对象，为每个物体配置所有顶点属性很快就变成一件麻烦事

#### 顶点数组对象(Vertex Array Object, VAO)



## Shaders



## Textures



## Transformations



## Coordinate Systems



## Camera



## Review



# Lighting



# Model Loading



# Advanced OpenGL



# Advanced Lighting



# PBR



# In Practice



# Guest Articles



# Code repository



