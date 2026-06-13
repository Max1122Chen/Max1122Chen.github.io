# Ogre源码解读

## RHI Abstraction

### VertexBuffer

file: OgreHardwareVertexBuffer.h



### VertexElement

VertexElement相当于描述了一个顶点属性



### VertexDeclaration

存储了一组VertexElement



### VertexData

在RenderSystem执行渲染操作（__render( )）时，VertexDeclaration和VertexBufferBinding会被一起打包在VertexData中，再包含在RenderOperation中，传递给 --render( )指向渲染错做



## OpenGL

#### VertexArrayObject

VAO是OpenGL中一个比较特殊的东西，其相当于向GPU表明了顶点的属性布局，Ogre中将其派生自VertexDeclaration，也就是规定VertexBuffer信息的抽象