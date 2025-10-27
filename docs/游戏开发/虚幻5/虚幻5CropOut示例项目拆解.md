# 虚幻5CropOut示例项目拆解

## 交互系统

​	该项目交互系统主要涉及到了“可建造Actor”（Building）和“可采集资源Actor”（Resourse）两类，他们派生于一个基类“BP_Interactable"，类如其名，它们是该游戏中主要需要互动的对象。

​	关于”Resource“类，有实体的树木啊，石头啊，浆果丛啊都有一个基类，而它们自己没有任何自己的逻辑，连接口都没有额外实现，全部由BP_Resoure基类已有的逻辑实现。它们的的网格体、数量、采集时间、单次采集量，全部在类默认值手动指定。

​	这种建立一个基类（甚至逻辑上不止一个基类）的做法不仅减少了重复编写逻辑的工作，使得整个可交互类清晰明了，还很方便用GetAllActorsofClass来遍历以方便保存，因为所有这些可交互的对象都派生自BP_Interactable。



## 处理游戏规则的类

### GameInstance

​	在该项目中，GameInstance扮演的一个惯常的角色就是负责保存和加载数据，它的行为有SaveGameToSlot，Save当前游戏中的资源，Save场上的Villagers等。

### GameMode

​	在GameMode中管理了游戏中的地图的初始化的一些内容，例如在新游戏开始时**随机**创建地图上的资源点，**随机**寻找点位创建玩家的Town Hall，创建玩家初始的3个Villagers。

​	此外GameMode还管理了玩家进入或退出游戏的一些逻辑。

​	GameMode还会管理当前玩家所拥有的“资源”的增加，并将其转发给GameInstance以存储。

#### 随机生成资源

​	在游戏开始时会随机生成资源点，这项任务是由一个Actor——BP_Spawner完成的，它通过自己的函数SpawnAsset来完成资源的随机生成。

​	它随机生成资源的过程分为三步：

1. 选取资源“群系”的生成点
2. 在“群系”范围内寻找合适的生成点
3. 生成出资源Actor

![image-20250211141653229](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211141653229.png)

![image-20250211141716633](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211141716633.png)

![image-20250211141824420](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211141824420.png)

## Common UI 的实践

​	**TODO Later**



## Villagers与它的AI

​	Villager被设计成可以根据当前玩家要求它去执行的工作而改变自身行为和外观，当它们开始被要求从事不同的Job，就会激活Event Change Job，这个函数会从一张DataTable中根据传入的NewJob（FName）去找对应的表格行结构，然后更新给Villager。

### Villager更换职业的逻辑

![image-20250211143631459](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211143631459.png)

​	值得注意的是，正如这里提到的，DataTable中存储的都是Asset的软引用（TSoftObjectPtr），开发者对此是做出了一定的优化。改变Villager的行为的方式就是给其更换行为树，所以其实Villager是有多棵行为树去定义其行为逻辑的。

![image-20250211143927737](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211143927737.png)

​	之后就是给Villager换上不同的工作对应的“工具"”帽子“以及”工作动画“等等，没什么好说的。

![image-20250211144004425](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211144004425.png)

#### Villager更换职业的接口

​	玩家通过这个接口来要求Villager更换其职业。

![image-20250211144737053](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211144737053.png)

#### Villager播放不同的动画蒙太奇

​	此处定义了一些Villager在应当进行不同的活动时的一些动画蒙太奇以及手中的工作的显示或隐藏，主要就是定义了一下Villager的行为的视觉呈现。![image-20250211145230525](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211145230525.png)

![image-20250211145517948](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211145517948.png)

![image-20250211145540527](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250211145540527.png)