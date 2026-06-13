# 虚幻Gamplay框架



![img](https://pic2.zhimg.com/v2-7a4d02470df2b433e0a0f5f3cb5701d1_r.jpg)



## World和Level

WorldSetting是一个Actor！原来如此吗？WorldSetting其实设置的是level。

关卡蓝图也是一个Actor！

World由Level组成，World至少有一个Level

虚幻中的“world”并非只是游戏世界这么简单，编辑器，测试窗口中的世界都是World，World具体是哪个由WorldContext管理，WorldContext是唯一的，由GameInstance管理。

> 因为经常有初学者会问到：我的Level切换了，变量数据就丟了，我应该把那些数据放在哪？再清晰直白一点，GameInstance就是你不管Level怎么切换，还是会一直存在的那个对象！



## Component

​	Component实现“功能”，而不实现具体业务逻辑，这些功能不依赖于特定游戏，是共通的



## Pawn

> ​	要非常清楚一点的是，Actor是我们用来表示3D游戏中对象的，所以Pawn继承于Actor，概念的重点是在于更清楚的去表示，而不是重点在于Pawn被当作逻辑的载体，就像棋子本身只能简单的表达出出个棋子，但是该如何走还是得再靠外部的Controller机制。你也可以想象成提线木偶，那个木偶就是Pawn，而提线的是Controller。Pawn表达的最关键点是可被玩家操纵的能力。

> 有些人一开始的时候会困惑应该选择Pawn还是Character，其实从[继承体系](https://zhida.zhihu.com/search?content_id=1514500&content_type=Article&match_order=1&q=继承体系&zhida_source=entity)中就可以了解到Character只不过是Pawn的加强特化版本。一般来说，如果你控制的角色是人形的带骨骼的，那就选择Character吧。

## Controller

​	Controller是UE Gameplay逻辑中最最重要的载体

> UE从Actor中分化了一些专门可供玩家“控制”的Pawn，那我们这篇就专门来谈谈该怎么个控制法！
> 所谓的控制，本质指的就是我们游戏的业务逻辑。比如说玩家按A键，角色自动找一个最近的敌人并攻击，这个自动寻找目标并攻击的逻辑过程，就是我们所谈的控制。



> 设计模式的本质就是[抽象变化](https://zhida.zhihu.com/search?content_id=1578067&content_type=Article&match_order=1&q=抽象变化&zhida_source=entity)。如果依照纯朴的"程序=数据+算法"的结构来看，再算上用于用户显示和输入的界面，那么就得到“程序=数据+算法+显示”。这三大基本块（数据，算法，显示）构成了程序的三大变化，而如何把这三者“+”到一起，用的就是我们的种种设计框架模式。
> 典型的，对于游戏：
>
> - “显示”指的是游戏的UI，是屏幕上显示的3D画面，或是手柄上的输入和震动，也可以是VR头盔的镜片和定位，是与玩家直接交互的载体；
> - “数据”指的是Mesh，Material，Actor，Level等各种元素组织起来的内存数据表示；
> - “算法”可以是各种渲染算法，[物理模拟](https://zhida.zhihu.com/search?content_id=1578067&content_type=Article&match_order=1&q=物理模拟&zhida_source=entity)，AI寻路，本文咱们就先暂时特指游戏开发者们编写的游戏业务逻辑。
>
> 抽象这三个变化，并归纳关系，就是典型的[MVC模式](https://zhida.zhihu.com/search?content_id=1578067&content_type=Article&match_order=1&q=MVC模式&zhida_source=entity)了

### MVC模式

MVC模式是指一种架构模式，它将软件程序分为三个部分：模型（Model）、视图（View）和控制器（Controller）。在原文的语境中，作者使用了MVC模式来描述游戏开发中的不同组成部分。具体来说：

- 模型（Model）：指的是游戏中的数据部分，如Mesh、Material、Actor、Level等元素组成的内存数据表示。
- 视图（View）：指的是游戏的用户界面，即屏幕上显示的3D画面，或是手柄上的输入和震动，也可以是VR头盔的镜片和定位。
- 控制器（Controller）：负责处理用户的交互操作，并根据这些操作更新模型的数据。

这种模式的优点包括：

- 低耦合性：各个组件之间的依赖较少，可以独立修改而不影响其他部分。
- 有利于开发分工：不同团队可以专注于不同的模块，提高开发效率。
- 组件复用：特别是视图部分，可以在多个项目中重复使用相同的代码或界面设计。
- 可维护性好：由于各部分职责明确，因此更容易理解和维护。

然而，对于复杂大型的游戏，仅仅依靠MVC模式可能不足以满足所有需求。游戏开发往往需要更复杂的逻辑框架来支持各种特定的游戏机制和功能，例如物理模拟、AI寻路等。因此，在实际应用中，开发者可能会在此基础上进行扩展或采用其他更适合的设计模式。



### Controller的大手

​	实际上，我们往往把Controller当作是“依附”在一个Pawn上，而实际上Controller才是真正的大佬，Pawn只是Controller手底下的小兵。

​	UE中，原生Controller与Pawn是1对1的关系，对于RTS类游戏来说，需要扩展原生Controller。

> **思考：Controller的位置有什么意义？**
> 既然Controller本身只是控制者，那它在场景中的位置和移动有什么意义吗？Controller为何还需要个SceneComponent?意义在于如果Controller本身有[位置信息](https://zhida.zhihu.com/search?content_id=1578067&content_type=Article&match_order=1&q=位置信息&zhida_source=entity)，就可以利用该信息更好的控制Pawn的位置和移动。



#### 哪些逻辑应该写在Controller中？

> 如同当初我们在思考Actor和Component的逻辑划分一样，我们也得要划分哪些逻辑应该放在Pawn中，哪些应该放在Contrller中。上文我们也说过，Pawn也可以接收用户输入事件，所以其实只要你愿意，你甚至可以脱离Controller做一个特立独行的Pawn。那么在那些时候需要Controller？哪些逻辑应该由Controller掌管呢？可以从以下一些方面考虑：
>
> - 从概念上，Pawn本身表示的是一个“能动”的概念，重点在于“能”。而Controller代表的是动到“哪里”的概念，重点在于“方向”。所以如果是一些Pawn**本身固有的能力逻辑**，如前进后退、播放动画、[碰撞检测](https://zhida.zhihu.com/search?content_id=1578067&content_type=Article&match_order=1&q=碰撞检测&zhida_source=entity)之类的就完全可以在Pawn内实现；而对于一些可替换的逻辑，或者智能决策的，就应该归Controller管辖。
> - 从对应上来说，如果一个逻辑只属于某一类Pawn，那么其实你放进Pawn内也挺好。而如果一个逻辑可以应用于多个Pawn，那么放进Controller就可以组合应用了。举个例子，在战争游戏中，假设说有坦克和卡车两种战车（Pawn），只有坦克可以开炮，那么开炮这个功能你就可以直接实现在坦克Pawn上。而这两辆战车都有的自动寻找攻击目标功能，就可以实现在一个Controller里。
> - 从存在性来说，Controller的生命期比Pawn要长一些，比如我们经常会实现的游戏中玩家死亡后复活的功能。Pawn死亡后，这个Pawn就被Destroy了，就算之后再Respawn创建出来一个新的，但是Pawn身上保存的变量状态都已经被重置了。所以对于那些需要**在Pawn之外还要持续存在的逻辑和状态**，放进Controller中是更好的选择。



### APlayerState

> 整个游戏世界构建起来就是为了玩家服务的，而玩家在游戏过程中，肯定要存取产生一些状态。而Controller作为游戏业务逻辑最重要的载体，势必要和玩家的状态打交道。所以Controller如果可以动态存取玩家的状态就会大为方便了。

APlayerState继承自Ainfo。

> **思考：哪些数据应该放在PlayerState中？**
> 从应用范围上来说，PlayerState表示的是玩家的游玩数据，所以那些关卡内的其他游戏数据就不应该放进来（GameState是个好选择），另外Controller本身运行需要的临时数据也不应该归PlayerState管理。而玩家在切换关卡的时候，APlayerState也会被释放掉，所有PlayerState实际上表达的是当前关卡的玩家得分等数据。这样，那些**跨关卡的统计数据**等就也**不应该**放进PlayerState里了，应该放在外面的GameInstance，然后用SaveGame保存起来。

### APlayerContoller

![img](https://pic3.zhimg.com/v2-88131e55febd8886e0f3c87b92c526e8_r.jpg)

> - Camera的管理，目的都是为了控制玩家的视角，所以有了PlayerCameraManager这一个关联很紧密的摄像机管理类，用来方便的切换摄像机。PlayerController的ControlRotation、ViewTarget等也都是为了更新Camera的位置。因为跟Camera的关系紧密，而Camera最后输出的是屏幕坐标里的图像，所以为了方便一些拾取的HitResult函数也都是实现在这里面。渲染章节会再详细介绍UE的摄像机管理。
> - Input系统，包括构建InputStack用来路由输入事件，也包括了自己对输入事件的处理。所以包含了UPlayerInput来委托处理。
> - UPlayer关联，既然顾名思义是PlayerController，那自然要和Player对应起来，这也是PlayerController最核心的部分。一个UPlayer可以是本地的LocalPlayer，也可以是一个网络控制UNetConnection。PlayerController只有在SetPlayer之后，才可以开始正常工作。
> - HUD显示，用于在当前控制器的摄像机面前一直显示一些UI，这是从UE3迁移过来的组件，现在用UMG的比较多，等介绍UI模块的时候再详细介绍。
> - Level的切换，PlayerController作为网络里通道，在一起进行Level Travelling的时候，也都是先通过PlayerController来进行RPC调用，然后由PlayerController来转发到自己World中来实际进行。
> - Voice，也是为了方便网络中语音聊天的一些控制函数。

> UE的思想是具象化一个“玩家实体”，并把所有的跟该玩家相关的操作和接口都交给它完成。一般其他的游戏引擎只是个“功能引擎”，提供了一些图形渲染UI系统等组件，但是在GamePlay这个层次就都非常欠缺了，一般都需要开发者自己搭建一套。而回想你写过的游戏，是不是也往往有一个Player类（一般是单件或者全局变量）？里面几乎是放着所有跟该玩家相关的业务逻辑代码。UE里的PlayerController就是这种概念，优点当然是直接方便好理解，缺点也如你所见，会代码膨胀得比较快。不过目前来说还算能接受，等某一块功能真的比较大了之后，可以再把它抽出一个单独的类来，如PlayerInput和PlayerCameraManager一样。

> 根据[单一职责原则](https://zhida.zhihu.com/search?content_id=1646070&content_type=Article&match_order=1&q=单一职责原则&zhida_source=entity)，我们写在哪个类里的变量应该尽量只符合该类的作用，所以PlayerController里的变量的意义在于更好的实现控制。比如假设玩家在一个关卡内可以按AABB来作弊获得100金币，但是限最多3次。那么这个按键的响应就应该由PlayerController来接收，然后调用AddCoin(100)，并更新PlayerController里的成员变量CoinCheatCount。也或者想实现马里奥的加速跑，也可以在PlayerController里增加Speed的成员变量。



## GameMode

> - 游戏或玩家的节奏，游戏可以分成一个个阶段，马里奥里的关卡就是一个阶段，而RPG游戏的一个大地图也是一个阶段。一个游戏也可能只有一个阶段，比如一直在宇宙里漫游的游戏。通常一个阶段结束后，会有一个结算，阶段之间，玩家也能明显感觉到切换感。
> - 游戏的机制，有时候即使是同样的场景，玩家却也能感觉就像在玩两个不同的游戏，比如MOBA里的同一张地图上的各种不同挑战模式。
> - 游戏的资源划分，有时候也能遇见同一个玩法应用在不同的场景上，比如赛车游戏的不同跑道。有时候也会在游戏的大地图里从酷热的沙漠到寒冷的极地。游戏开发中也总是倾向于给游戏用到的资源划分成组的进行载入和释放。
>
> 通过以上的分析，也和以前的一贯思路一样，我们发现在思考“关卡”这件事情上，也是要保持头脑清晰的分清“表示”和“逻辑”。玩法就是“逻辑”，场景就是“表示”。所以我们如果以逻辑来划分游戏，得到的就是一个个World的概念；如果以表示来划分，得到就是一个个Level。一场游戏中，玩法再复杂但也只有一个，场景却可以无限大，所以可以有很多个表示[拼接组装](https://zhida.zhihu.com/search?content_id=1669170&content_type=Article&match_order=1&q=拼接组装&zhida_source=entity)，因此是World包含Level，而不是反过来。现在回过头来回想一下Cocos2dx和Unity的世界观，它们的概念还只是在表示层，在游戏实例和关卡之间少了一个更高级的逻辑概念。



> 按照UE的设计理念和经过Controller的经历，我想我也不用多解释了从Actor再派生出一个WorldController的方式了，可以直接的享受Actor已经提供的一切福利。一个World的Controller想不出有什么需要展示渲染的，因此可以直接从AInfo派生吧。哦，WorldController是我瞎编的，在UE3里它叫做GameInfo，到了UE4它改名为了GameMode。笼统的讲，一个World就是一个Game，把玩法叫做Mode，我们应该也能接受吧。

<img src="https://pic4.zhimg.com/v2-df7c0a12f2b806dbb60e84a95ae9758d_r.jpg" alt="img" style="zoom: 67%;" />

> 既然勇敢的承担了游戏逻辑的职责，说他是AInfo家族里的扛把子也不为过，因此GameMode身为一场游戏的唯一逻辑操纵者身兼重任，在功能实现上有许多的接口，但主要可以分为以下几大块：
>
> 1. **Class登记**，GameMode里登记了游戏里基本需要的类型信息，在需要的时候通过UClass的反射可以自动Spawn出相应的对象来添加进关卡中。前文说过的Controller的类型登记也是在此，GameMode就是比Controller更高一级的领导。
>
> ![img](https://pic1.zhimg.com/v2-42d75bd239bcad7995349a712f1ba37e_1440w.png)
>
> 
>
> 1. **游戏内实体的Spawn**，不光登记，GameMode既然作为一场游戏的主要负责人，那么游戏的加载释放过程中涉及到的实体的产生，包括玩家Pawn和PlayerController，AIController也都是由GameMode负责。最主要的SpawnDefaultPawnFor、SpawnPlayerController、ShouldSpawnAtStartSpot这一系列函数都是在接管玩家实体的生成和释放，玩家进入该游戏的过程叫做Login（和服务器统一），也控制进来后在什么位置，等等这些实体管理的工作。GameMode也控制着本场游戏支持的玩家、旁观者和AI实体的数目。
> 2. **游戏的进度**，一个游戏支不支持暂停，怎么重启等这些涉及到游戏内状态的操作也都是GameMode的工作之一，SetPause、ResartPlayer等函数可以控制相应逻辑。
> 3. **Level的切换**，或者说World的切换更加合适，GameMode也决定了刚进入一场游戏的时候是否应该开始播放开场动画（cinematic），也决定了当要切换到下一个关卡时是否要bUseSeamlessTravel，一旦开启后，你可以重载GameMode和PlayerController的GetSeamlessTravelActorList方法和GetSeamlessTravelActorList来指定哪些Actors不被释放而进入下一个World的Level。
> 4. **多人游戏的步调同步**，在多人游戏的时候，我们常常需要等所有加入的玩家连上之后，载入地图完毕后才能一起开始逻辑。因此UE提供了一个MatchState来指定一场游戏运行的状态，意义看名称也是不言自明的，就是用了一个[状态机](https://zhida.zhihu.com/search?content_id=1669170&content_type=Article&match_order=1&q=状态机&zhida_source=entity)来标记开始和结束的状态，并触发各种回调。

> **思考：哪些逻辑应该写在GameMode里？哪些应该写在Level Blueprint里？**
> 我们依旧要问这个老土的问题。根据我们前面的知识，我们知道每个Level其实也是有自己的LevelScriptActor的，那么这两个有什么区别？可以从这几个方面来回答：
>
> - 概念上，Level是表示，World是逻辑，一个World如果有很多个Level拼在一起，那么也就是有了很多个LevelScriptActor，无法想象在那么多个地方写一个完整的游戏逻辑。所以GameMode应该专注于逻辑的实现，而LevelScriptActor应该专注于本Level的表示逻辑，比如改变Level内某些Actor的运动轨迹，或者某一个区域的重力，或者触发一段特效或动画。而GameMode应该专注于玩法，比如胜利条件，怪物刷新等。
> - 组合上，同Controller应用到Pawn一样道理，因为GameMode是可以应用在不同的Level的，所以通用的玩法应该放在GameMode里。
> - GameMode只在Server存在（单机游戏也是Server），对于已经连接上Server的Client来说，因为游戏的状态都是由Sever决定的，Client只是负责展示，所以Client上是没有GameMode的，但是有LevelScriptActor，所以GameMode里不要写Client特定相关的逻辑，比如操作UI等。但是LevelScriptActor还是有的，而且支持RPC，即使如此，LevelScriptActor还是应该只专注于表现，比如网络中触发一个特效火焰。至于UI，可以通过PlayerController的RPC，然后转发到GameInstance来操作。
> - 跟下层的PlayerController比较，GameMode关心的是构建一个游戏本身的玩法，PlayerController关心的玩家的行为。这两个行为是独立正交可以自由组合的。所以想想哪些逻辑属于游戏，哪些属于玩家，就应该清楚写在哪里了。
> - 跟上层的GameInstance比较，GameInstance关注的是更高层的不同World之间的逻辑，虽然有时候他也把手伸下来做些UI的管理工作，不过严谨来说，在UE里UI是独立于World的一个结构，所以也还算能理解。因此可以把不同GameMode之间协调的工作交给GameInstance，而GameMode只专注自己的玩法世界。

### GameState

> 上回说到了APlayerState用来保存玩家的游戏数据，那么同样的，对于一场游戏，也需要一个State来保存当前游戏的状态数据，比如任务数据等。跟APlayerState一样，GameState也选择从AInfo里继承，这样在网络环境里也可以Replicated到多个Client上面去。
>
> <img src="https://picx.zhimg.com/v2-336fd667fb674bf176fa198741eec129_1440w.png" alt="img" style="zoom: 67%;" />
>
> 比较简单，第一个MatchState和相关的回调就是为了在网络中传播同步游戏的状态使用的（记得GameMode在Client并不存在，但是GameState是存在的，所以可以通过它来复制），第二部分是玩家状态列表，同样的如果在Client1想看到Client2的游戏状态数据，则Client2的PlayerState就必须广播过来，因此GameState把当前Server的PlayerState都收集了过来，方便访问使用。
> 关于使用，开发者可以自定义GameState子类来存储本GameMode的运行过程中产生的数据（那些想要replicated的!），如果是GameMode游戏运行的一些数据，又不想要所有的客户端都可以看到，则也可以写在GameMode的成员变量中。重复遍，PlayerState是玩家自己的游戏数据，GameInstance里是程序运行的全局数据。

## UPlayer

> 让我们假装自己是UE，开始编写Player类吧。为了利用上UObject的那些现有特性，所以肯定是得从UObject继承了。那能否是AActor呢？Actor是必须在World中才能存在的，而Player却是比World更高一级的对象。玩游戏的过程中，LevelWorld在不停的切换，但是玩家的模式却是脱离不变的。另外，Player也不需要被摆放在Level中，也不需要各种Component组装，所以从AActor继承并不合适。那还是保持简单吧：
>
> <img src="https://pica.zhimg.com/v2-f1fee0eaf2b5fea87085f71895f4e730_r.jpg" alt="img" style="zoom:80%;" />

### ULocalPlayer

> 然后是本地玩家，从Player中派生下来LocalPlayer类。对本地环境中，一个本地玩家关联着输入，也一般需要关联着输出（无输出的玩家毕竟还是非常少见）。玩家对象的上层就是引擎了，所以会在GameInstance里保存有LocalPlayer列表。
>
> ![img](https://pic4.zhimg.com/v2-0cdc26e0b66ec808197a76cb8b09350d_1440w.png)

## GameInstance

> 一路向上，依照设计里一个最朴素的原理：自己是无法创建管理自身的，所以Player也需要一个创建管理和存储的地方。另一方面，上文提到Player固然可以负责一些跟玩家相关的业务逻辑，但是对于World之上协调管理的逻辑却也仍然无处安放。

> 简单的事情就不用多讲了，UE提供的方案是一以贯之的，为我们提供了一个GameInstance类。为了受益于UObject的反射创建能力，直接继承于UObject，这样就可以依据一个Class直接动态创建出来具体的GameInstance子类。
>
> ![img](https://pica.zhimg.com/v2-e3572a2d46608304109104e6174688d8_1440w.png)

​	一般来说，GameInstance只有一个

> ![img](https://pica.zhimg.com/v2-94d1f4e3750b6f4fd09d02b20bc980b0_1440w.png)

> **思考：哪些逻辑应该放在GameInstance？**
> 第二个惯例的问题是，这一层应该写些什么逻辑。顾名思义，既然是作为游戏中全局唯一的长者，我们就应该给他全局的控制权。在逻辑层面，GameInstance往下看是：
>
> 1. Worlds，Level的切换实际发生地是Engine，而GameInstance可以说是UE之神其下的唯一代言人，所以GameInstance也可以代之管理World的切换等。我们可以在GameInstance里实现各种逻辑最后调用Engine的OpenLevel等接口。
> 2. Players，虽然一般来说我们直接控制Players的机会不多，都是配置好了就行。但要是到了需要的时候，GameInstance也实现了许多的接口可以让你动态的添加删除Players。
> 3. UI，UE的UI是另一套World之外的系统，虽然同属于Viewport的显示之下，但是控制结构跟Actor们并不一样。所以我们常常会需要控制UI各种切换的业务逻辑，虽然在Widget的Graph里也可以写些简单的切换，但是要想复用某些切换逻辑的时候，在特定的Wdiget里就不合适了，而GameMode一方面局限于Level，另一方面又只存在于Server；PlayerController也是会切换掉的，同时又只存在于World中，所以最后比较合适的就剩下GameInstance了，以后当然有可能了可能会扩展出个UI的业务逻辑Manger类，不过那是后话了。
> 4. 全局的配置，也常常需要根据平台改变一些游戏的配置，Execute一些ConsoleCommand，GameInstance也是这些命令的存放地。
> 5. 游戏的额外第三方逻辑，如果你的游戏需要其他一些控制，比如自己写的网络通信、自定义的配置文件或者自己的一些程序算法，如果简单的话，GameInstance也可以一放，等复杂起来了，也可以把GameInstance当作一个模块容器，你可以在里面再扩展出来其他的子逻辑模块。当然如果是插件的话，还是在自己的插件Module里面自行管理逻辑，然后把协调工作交给GameInstance来做。

### SaveGame

> 关于存档数据关联的逻辑，再重复几句，对于那些需要直接在全局处理的数据逻辑，也可以直接在SaveGame中写方法来实现。比如实现AddCoin接口，对外隐藏实现，对内可以自定义附加一些逻辑。USaveGame可以看作是一个全局持久数据的业务逻辑类。跟GameInstance里的数据区分就是，GameInstance里面的是临时的数据，SaveGame里是持久的。清晰这一点区分，到时就不会纠结哪些属性放在哪里，哪些方法实现在哪里了。