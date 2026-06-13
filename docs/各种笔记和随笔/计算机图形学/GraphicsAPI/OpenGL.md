# OpenGL

OpenGL的运作方式是一个“状态机”

## Some concepts

### Vertex Buffer

只是一个Buffer，一块在显存中的内存

OpenGL用ID（unsigned int）来唯一标识Buffer

Buffer仅仅只是用来**存有顶点**，是顶点数据的缓存，而不会告诉GPU如何“认知”这些顶点，例如哪些顶点构成了一个三角形

例如我们有一个装有6个float的数组，我们知道它表示三个顶点（3个二维坐标），但是GPU并不知道

对Buffer来说，它存着的东西只是一些字节，我们还需要告诉GPU如何认知Buffer中的内容，这需要一个VertexAttributePtr

> [!IMPORTANT]
>
> 需要澄清的是，Vertex不只是一个“位置”，还可以包含了“颜色”，“纹理”，“法向”等等信息



值得注意的是，glBindBuffer( GL_XXX_BUFFER, ID )函数的意义是让状态机知道：现在开始对GL_XXX_BUFFER的操作都是针对ID指向的那个Object。这是理解OpenGL的一个关键。

包括对VAO也是这个逻辑。不过因为VAO就只有一种VertexArrayObject，所以没有像VBO的类型参数

### Vertex Attribute Pointer

Vertex Attribute Buffer Pointer为GPU如何理解Vertex Buffer中的字节



stride 指的是vertex之间的距离

ptr 指的是对于“顶点”，它的某个属性在一个顶点内的偏移量



值得注意的是：

绑定VAO，VBO时的顺序非常关键

VAO相当于只是一个记录了VBO与属性指针之间关系的“记录员”，它并没有记住VBO是怎样的或属性指针是怎样的，而只是记录了一种关系

创建并bind VAO后，必须先bind VBO，再配置并启用属性指针

glVertexAttribPointer的逻辑是，把当前GL_ARRAY_BUFFER中绑定的VBO和程序员提供的规则建立关联，而此时激活的VAO会在执行glVertexAttribPointer时记录下这个关联关系

如果不先绑定VBO，就相当于GL_ARRAY_BUFFER中啥也没有/用了之前绑定的东西，这就导致了错误



### Shader

Shader用于告诉GPU要对现在所使用的Buffer中的Vertices做出怎样的事情

Shader会在drawcall之后开始起作用

> [!NOTE]
>
> 有些GPU驱动提供了默认的着色器，因此如果一开始不定义任何着色器也可能渲染出结果

#### Vertex Shader







#### Fragment Shader

Fragment Shader决定一个像素的颜色



### 使用Shader

使用shader首先需要编写shader的代码，接着把它编译好，最后将它Attach到一个Program上，再把Program link好，这样才可以真正使用它

当Shader被链接好了之后，就可以将它delete掉



Shader的创建需要一段源代码，它会被通过字符串的形式传给glComplieShader



我们可以单独使用一些文件来存储Shader的源代码，然后写一些工具函数来将文件转换成C风格字符串



### Index Buffer

在实际情况中，很多顶点实际上会被重复使用，如果把重复使用到的顶点存放在Vertex Buffer中，就会有点浪费内存，尤其是当”顶点“的属性不止”位置“时

这时我们可以通过定义另一个数组Index Buffer来指导GPU以怎样的顺序访问Vertex Buffer



### Uniform

uniform用来从CPU向向shader传递数据，吗？



### Vertex Array

Vertex Array 存储了Vertex Buffer和顶点布局信息（也就是Vertex Attribute Ptr）的对应信息，它会把当前绑定着的Vertex Buffer和指定了一个特定的下标的Vertex Attribute Ptr 建立关联关系，这样就可以通过Vertex Array下标来选择使用不同的Vertex Buffer，而这些Vertex Buffer的Vertex Attribute Ptr是被关联好了的







## Deal with errors in OpenGL

当我们写错了一些东西的时候，OpenGL给我们的反馈可能仅仅只是一个黑色的窗口（什么都没有），我们希望能够了解到OpenGL遇到的错误

一种角度是，程序员在某些地方用glGetError来向OpenGL主动地查询错误

另一种角度是用glMessageErrorCallBack绑定一个回调函数，当OpenGL遇到问题时，主动提出



使用glGetError仍然没有那么好用，因为它只会告诉你出了什么错，而没办法直接定位它并且像异常那样打断程序

我们可以使用一些宏来帮助我们



## Draw Function in OpenGL

OpenGL的Draw相关的函数无需依赖VBO，只需依赖VAO，因为VAO已经明确了某些VBO和其对于的顶点布局信息的关系了







## Abstract OpenGL into classes

