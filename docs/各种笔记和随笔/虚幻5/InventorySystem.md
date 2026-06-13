# InventorySystem

## 简介

通过创建plugin构建可插拔的InventorySystem，可复用在其他项目中。



## 起始工作

1. 为Inventory插件增加自定义类型的Log
2. 在插件内创建一个PlayerController，用来增加玩家的输入映射
3. 给PlayerController增加一个IA，IMC，将其进行绑定，并在编辑器中指定
4. 制作一个简单的HUD用来作为屏幕中心的瞄准标，方便与场景物体互动



## Trace Item

### Trace Item Channel

为了提高trace的效率，可以自定义一个单独的Trace Channel和Collision Profile，把可以被采集的物品的碰撞预设设成自定义的Profile，提高检测效率。

### Do Trace

做检测的原理很简单，就是做射线检测，获取屏幕中心，并将其通过逆投影获得其指向的世界坐标和指向的世界方向，以此可以进行单通道的射线检测（图形学知识）。

值得注意的是，在保留对Hit到的Actor的引用时（Hit到的只能是接受ItemTrace通道的Actor），需要用WeakPtr来避免干扰垃圾回收。



### Show/Hide Pickup Message & Item Comp

我们可以在玩家屏幕指针检测到可拾取物体时将物品信息展示出来，为此，我们可以先定义一个Item Component来使得一些Actor成为可拾取的物体，以此赋予它们Pickup Message。

并且我们可以为其加上玩家视线聚焦时将其高亮，失焦时取消高亮的效果，用接口做就行。



## Inventory Component

接下来制作Inventory Component，即管理实际库存的组件。我们会把它挂在PC上，因为PC仅存在于服务器和对应的玩家的客户端上，而不会存在于别的玩家的客户端上。

Inventory Component负责库存系统的Data管理，同时直接控制UI视觉表现。

按理来说Inventory Component不仅仅可由玩家拥有，也可以挂在其他Actor上，例如“容器”，这样发挥了其可复用性，也使得游戏中的库存系统逻辑是一致的，但是课程中并没有这么做。InventoryComponent本身和Inventory的UI做得也比较高耦合度，感觉不是很好，最好还是要分离Data和View。



### Fast Array Serializer

尽管Inventory的逻辑结构可以简单化为一个数组（在虚幻4 RPG示例中使用了TMap实现），但只考虑如何实现还不够，多人游戏中的复制也是一个需要考虑的方面。

粗暴一点，每当物品栏发生变化，就把物品栏中的所有东西全部复制一遍，这样太浪费了。

Fast Array Serializer是一个用来实现增量复制的数组（其实它本身不是，你仍然要在FAS中声名一个TArray来实现真正的Array），能够提高性能。

需要配合使用一个继承了FastArraySerializerItemFStruct作为FAS的数组元素

FAS规定了一些接口函数，不是必须实现的，不是虚函数，但是可以实现它们来增加FAS的功能。具体可以看FAS的源码中的说明。

可以在PostReplicatedAdd和PreReplicatedRemove中加上向外广播Inventory变化的委托

最后还需要一个类型萃取来说明我们的自定义FAS需要被增量复制（DeltaReplicate）



### AddEntry & RemoveEntry

课程有一个值得注意的点是，只允许在服务器上执行AddEntry，因此在逻辑中需要鉴权。

使用FAS的一个要点是需要在增删时把TArray的对应Entry标记为Dirty以便告知FAS哪些东西是需要被网络复制的。

由于你不一定能够添加物品成功，所以在InventoryManagerComponent中调用的函数应该是TryAddItem，尝试去做。

并且我们在InventoryComponent上实际上需要发RPC来让服务器执行权威的Add和Remove逻辑。