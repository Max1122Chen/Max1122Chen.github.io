# Unity RPG 教程跟学笔记

## 基础知识

改变Sprite的Order in Layer可以改变Sprite的遮挡关系

Ridgid Body决定GO拥有物理属性

Collider决定物体拥有碰撞

SerializeField可以把private的变量暴露成EditDefaultOnly（你懂我意思）

裁剪Sprite后，需要将调整合适的大小，并将compression调整为none，并将filter mode调成point

动画需要AnimatorController

Animator类似于动画蓝图

Gizmos.DrawLine可以单纯地画线但不做碰撞检测

Raycast是进行射线碰撞检测的函数

使用Blend Tree可以在同一个状态选择播放不同的动画，可以用来减少状态转换的次数

在C#中使用Array作为定长数组，且不能添加删除元素，使用List作为动态数组，可以添加删除

OverlapXXX可以对一个几何区域进行碰撞检测

动画可以直接加上Animation Event去调用一个GO上函数？

可以使用nameof(方法名)来给Invoke传递需要调用的方法名称来避免不一致。



### Make Timer

在Unity中，简单的"timer"是由开发者自己利用引擎内的时间变化来实现的，而不是像UE那样有写好的Timer。开发者既可以用Update + Time.deltaTime来通过每帧更新float timer去判断时间过去了多久。也可以只记录时刻，在下一次重复一样的行为时判断两个时刻之间的时间间隔是否满足条件。总之方式很灵活。



## Finite State Machine

有限状态机不仅可以方便管理在不同State下的行为，处理一些Input有效性的问题。同时也便于角色的行为扩展和维护。



我们需要自定义一个不继承其他类的类EntityState来作为State的基类和一个StateMachine类

EntityState基类只需要提供Enter，Exit，Update三个虚函数作为统一接口

StateMachine只需要定义Initialize和ChangeState函数









