# 使用UE开发游戏前置软件安装

> [!IMPORTANT]
>
> ​	**一定要逐字逐句看清楚本文的指导，不然可能会出抽象的问题！！！**

> [!IMPORTANT]
>
> ​	**一定要逐字逐句看清楚本文的指导，不然可能会出抽象的问题！！！**

> [!IMPORTANT]
>
> ​	**一定要逐字逐句看清楚本文的指导，不然可能会出抽象的问题！！！**



## **1.** 安装Visual Studio，Epic Game Launcher与虚幻引擎

（1）安装虚幻引擎，**参考教程**，B站->麦克斯大大->收藏->虚幻入门精选->《UE4初学者入门精选》，P4，26：52后进入下载教程，下载版本选择5.4（即5.4.4），最好不要按照默认下载路径，在C盘外合适的磁盘安装较好，可以新建文件夹“UnrealEngine”把各种版本的引擎放在其中统一管理较好。

> [!CAUTION]
>
> ​	初次打开UE时，一定要**慢慢地**把所有弹出的”允许应用访问某某某“的提示框一个一个好好地”允许“了，不然的话可能会出现抽象的问题导致启动不了含有C++代码的UE项目。

（2）安装VS，**参考教程**，B站->麦克斯大大->收藏->虚幻入门精选->《UE5C++零基础教程合集》->P2，下载VS时勾选C++游戏开发的UE启动程序应该会自动安装Epic Game Launcher（没试过这样安装，建议按后面的做），或者在Epic官网下载Epic Game Launcher，再下载虚幻引擎。

> [!CAUTION]
>
> ​	一定要好好对照教程要求安装的VS的组件安装，尤其是要注意安装这个东西：（在”编译器、生成工具和运行时“一栏下面）！！
>
> ![image-20250109230636165](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109230636165.png)
>
> ​	如果你在上面已经完成了Epic Game Launcher和虚幻引擎的安装，就不要勾选”Unreal Engine 安装程序“！！！



## **2.** 安装组件虚拟局域网工具TailScale

​	直接前往TailScale官网下载TailScale，**不需要注册账户**，下载完成后，需要连接到MaxChen的虚拟局域网。下载完成后会TailScale会自动在任务栏小图标中显示：（第一行最后一个九个小圆点的图标）

<img src="C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109231041347.png" alt="image-20250109231041347" style="zoom:50%;" />

​	最开始会显示“Starting”，但是由于网络的问题可能会变成“Failed to Connect to Network service"，要等之后成功连上服务才能开始连接到虚拟局域网。下图是连接好的状态。

<img src="C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109231130574.png" alt="image-20250109231130574" style="zoom: 67%;" />

​	连接的操作是，在tailscale处log in，选择以github的方式log in，账号是Max1122Chen@126.com，password是Max20051122.

## 3.安装版本管理工具Preforce Helix

参考教程：https://juejin.cn/post/7146486118605651976

（直接看我下文的教程更清晰）

下载教程：

首先在Preforce官网分别下载到这两个：**（先安装第二个，再安装第一个）**

由于Preforce上下载有时会很抽象，搞半天搞不出来下载页面，所以我提供了百度网盘资源：

通过百度网盘分享的文件：HelixCore
链接：https://pan.baidu.com/s/16JOaKCIo_anlO2VZN85Erg?pwd=1122 
提取码：1122

![image-20250109225715786](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225715786.png)

良好的文件结构：

![image-20250109225952019](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225952019.png)

###  下载HelixCoreServer

对Helix Core Server推荐下载路径：（注意看文件路径）

![image-20250109225703019](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225703019.png)

​	下图的Port Number无需更改，注意这里的P4ROOT，我把它放在了HelixCoreServer文件夹下一个新的文件夹“Server”，这样做仅仅是为了保持清楚的文件结构，免得文件混在一起。

![image-20250109225741677](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225741677.png)

​	Server无需更改，UserName默认填入的是电脑当前账户的名字，可以改一个自己喜欢的（最好能够清楚地表明你是谁），Text Editing App无需更改。

<img src="C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225750864.png" alt="image-20250109225750864" style="zoom:80%;" />

接下去直接安装（install）即可

###  下载HelixCoreApps

对Helix Core Apps（P4V）：

注意下载路径

![image-20250109225845442](C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225845442.png)

下一步填写和你在安装Helix Core Server时填一样的

<img src="C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225902017.png" alt="image-20250109225902017" style="zoom:80%;" />

下一步直接安装。

安装成功后：

<img src="C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225909901.png" alt="image-20250109225909901" style="zoom:80%;" />

点Exit默认Exit后自动打开P4V，会弹出这样一个窗口：

<img src="C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225926239.png" alt="image-20250109225926239" style="zoom:80%;" />

此时点击OK会显示No such User，只需在第二行New一个User就行，

<img src="C:/Users/Max1122Chen/AppData/Roaming/Typora/typora-user-images/image-20250109225934888.png" alt="image-20250109225934888" style="zoom:80%;" />

​	用户名最好能够清楚地表明你是谁，且和之前写的一致。之后再Browse User，选择你新建的User，点OK即可开始使用。之后直接退出即可，暂且不做下面的事情。

### 工具使用的逻辑：

​	首先P4S（HelixCoreServer）使得每一个使用这个工具的人都可以作为服务器来存储文件，作为服务器的人相当于一个权威的仓库，其他成员之后可以根据IP地址和开放端口号连接到指定的一台服务器上参与工作。

​	在最开始安装完成两个软件后，在刚刚上一步New User的时候，其实是你本机作为服务器，你自己及是客户端，又是服务器，你自己连上了你自己。在连接任何一个服务器时，都需要在服务器创建一个新的账号，就是New User，最好保持你在别人的服务器上的账号和你自己在你的服务器上的账号账号信息一样，方便你记住。

​	使用这个工具需要P2P联网，没有公网ip的话不用虚拟局域网找不到服务器，所以前面要连接到一个虚拟局域网，这样才能通过TailScale分配的虚拟ip找到对方。

## 4. 安装Rider

（对UE有适配的更方便的IDE，不写代码不用装，但是建议装，可以看源码，理解更全面）

​	在该网址下的”JetBrain“可以下载到Rider最新版（下面的下载网页中下载Rider时好像会自带下载JetBrain通用破解工具，一定要按照破解教程执行）和JetBrain通用破解工具，Rider需要破解使用。

[石用软件-专注分享实用软件 (shiyrj.top)](https://www.shiyrj.top/)