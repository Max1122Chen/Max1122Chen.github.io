# 虚幻与C++

## 官方入门内容

### Actor类——从CameraDirector入手



我们选择从Actor派生CameraDirector类



- 知识点：**UPROPERTY** 宏使得变量对 **虚幻引擎** 可见。

我们需要在类源文件中引入GameplayStatics.h，从而使用一些通用函数

#include "Kismet/GameplayStatics.h"



将按照UE官方文档实现后的CameraDirector拖入世界创造一个**实例**，会发现它无法移动，可以看到它的**Details**中无**transform**属性

此时的CameraDirector类类似一个管理类，或脚本



### 组件与碰撞

#### 前言——Pawn类的入门

**Pawn**继承自**Actor**，**Pawn**是一种由真实玩家或AI控制的 **Actor**。

在类定义中添加

```c++
 UPROPERTY(EditAnywhere)
     USceneComponent* OurVisibleComponent;
 //用于记录创建的组件

```

在构造函数中添加

```c++
// 创建可附加内容的虚拟根组件。
     RootComponent = CreateDefaultSubobject<USceneComponent>(TEXT("RootComponent"));
     // 创建相机和可见对象
     UCameraComponent* OurCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("OurCamera"));
     OurVisibleComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("OurVisibleComponent"));//静态网格组件
     // 将相机和可见对象附加到根组件。偏移并旋转相机。
     OurCamera->SetupAttachment(RootComponent);
     OurCamera->SetRelativeLocation(FVector(-250.0f, 0.0f, 250.0f));
     OurCamera->SetRelativeRotation(FRotator(-45.0f, 0.0f, 0.0f));
     OurVisibleComponent->SetupAttachment(RootComponent);
```

此后可在编辑器中绑定静态网格给MyPawn



#### 正题

​	在构造函数中添加将生成各种有用组件并将以层级进行安排的代码。将创建与物理场景交互的 **球体组件**、视觉显示碰撞形态的 **静态网格体组件**、可随意开关的**粒子系统组件**，及可用于附加 **摄像机组件** 控制游戏视角的 **弹簧臂组件**。在此之前，我们应该添加最终会用到的头文件。在头文件信息所在行的下方，我们要加入：

```c++
  #include "UObject/ConstructorHelpers.h"
  #include "Particles/ParticleSystemComponent.h"
  #include "Components/SphereComponent.h"
  #include "Camera/CameraComponent.h"
  #include "GameFramework/SpringArmComponent.h"
```

​	决定用作层级根节点的组件。球体组件是一种可与游戏场景交互并碰撞的物理实体，本教程将使用球体组件。注意：**Actor** 的层级中可拥有多个启用物理的组件

```c++
// 根组件将成为对物理反应的球体
USphereComponent* SphereComponent = CreateDefaultSubobject<USphereComponent>(TEXT("RootComponent"));
RootComponent = SphereComponent;
SphereComponent->InitSphereRadius(40.0f);
SphereComponent->SetCollisionProfileName(TEXT("Pawn"));
```

##### 弹簧臂组件设置

​	利用弹簧臂组件，可将摄像机以低于其追踪Pawn的速度加速和减速，从而获得更平滑的摄像机附加点。其同时拥有内置功能，可防止摄像机穿过固体对象，用于处理如玩家在第三人称游戏里退到角落时等情况。虽然弹簧臂组件并非必要的组件，但它能够轻松让摄像机在游戏中移动时变得更加平滑。

```c++
// 使用弹簧臂给予摄像机平滑自然的运动感。
USpringArmComponent* SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraAttachmentArm"));
SpringArm->SetupAttachment(RootComponent);
SpringArm->SetRelativeRotation(FRotator(-45.f, 0.f, 0.f));
SpringArm->TargetArmLength = 400.0f;
SpringArm->bEnableCameraLag = true;
SpringArm->CameraLagSpeed = 3.0f;
```

##### 摄像机组件设置

​	摄像机组件的创建十分容易，在这个示例中，你无需进行任何特殊设置。弹簧臂内置一个特殊的**插槽**，可供我们添加对象，这样就不必将对象直接添加到组件的根节点上。

```c++
 // 创建摄像机并附加到弹簧臂
UCameraComponent* Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("ActualCamera"));
Camera->SetupAttachment(SpringArm, USpringArmComponent::SocketName);
```

组件已创建并附加，现在应将此Pawn设为受默认玩家控制。只需下列代码即可：

```c++
// 控制默认玩家
AutoPossessPlayer = EAutoReceiveInput::Player0;

```

#### 配置游戏输入

共有两种输入映射类型：操作和轴。

- **操作映射** 适用于"是/否"输入，例如鼠标或手柄上的按钮。被按下、松开、双击或短时长按时，其将进行报告。跳跃、射击或与物体互动等**离散**操作是这类映射的理想对象。
- **轴映射** 是连续的，可将其视为"程度"输入，例如手柄上的摇杆，或者鼠标光标的位置。其会逐帧报告自身的值，即使未移动也进行报告。通常使用此方法处理如行走、四处查看和操纵车辆等有量级或方向的对象。



​	要实现输入映射绑定，需要在编辑器中指定

#### 直接在MyPawn上编写和绑定游戏操作

在源文件中添加：（头文件也要添加声明）

```c++
 void AMyPawn::Move_XAxis(float AxisValue)
     {
         // 以100单位/秒的速度向前或向后移动
         CurrentVelocity.X = FMath::Clamp(AxisValue, -1.0f, 1.0f) * 100.0f;
     }
 
     void AMyPawn::Move_YAxis(float AxisValue)
     {
         // 以100单位/秒的速度向右或向左移动
         CurrentVelocity.Y = FMath::Clamp(AxisValue, -1.0f, 1.0f) * 100.0f;
     }
 
     void AMyPawn::StartGrowing()//Grow绑定了space，作用是进行缩放
     {
         bGrowing = true;
     }
 
     void AMyPawn::StopGrowing()
     {
         bGrowing = false;
     }
```

​	使用 **FMath::Clamp** 约束输入中得到的值，将其约束在-1到+1的范围内。虽然本例中不存在此问题，但若有会对轴产生相同影响的多个键，则玩家同时按下此类输入时会将累加这些值。例如，如W键和向上方向键均映射到MoveX，且缩放均为1.0，同时按下这两个键会得到2.0的AxisValue。如不进行限制，玩家将以两倍速度移动。



现在已定义输入函数，接下来需进行绑定，以便对相应输入做出反应。将以下代码添加到 **AMyPawn::SetupPlayerInputComponent** 中：

```c++
// 在按下或松开"Grow"键时做出响应。
     InputComponent->BindAction("Grow", IE_Pressed, this, &AMyPawn::StartGrowing);
     InputComponent->BindAction("Grow", IE_Released, this, &AMyPawn::StopGrowing);
 
     // 对两个移动轴"MoveX"和"MoveY"的值逐帧反应。
     InputComponent->BindAxis("MoveX", this, &AMyPawn::Move_XAxis);
     InputComponent->BindAxis("MoveY", this, &AMyPawn::Move_YAxis);
 
```

变量现在将根据配置的输入进行更新。接下来只需编写代码使其完成部分操作。将以下代码添加到 **AMyPawn::Tick**：

```c++
// 根据"Grow"操作处理增长和缩减
     {
         float CurrentScale = OurVisibleComponent->GetComponentScale().X;
         if (bGrowing)
         {
             // 一秒内增长到两倍大小
             CurrentScale += DeltaTime;
         }
         else
         {
             // 以增长速度缩减一半
             CurrentScale -= (DeltaTime * 0.5f);
         }
         // 确保不会降至初始大小以下，或者增至两倍大小以上。
         CurrentScale = FMath::Clamp(CurrentScale, 1.0f, 2.0f);
         OurVisibleComponent->SetWorldScale3D(FVector(CurrentScale));
     }
 
     // 根据"MoveX"和"MoveY"轴处理移动
     {
         if (!CurrentVelocity.IsZero())
         {
             FVector NewLocation = GetActorLocation() + (CurrentVelocity * DeltaTime);
             SetActorLocation(NewLocation);
         }
     }
```

#### 用移动组件管理Pawn的移动

​	Pawn移动组件拥有部分强大内置功能，以便使用常见**物理功能**，同时便于在大量Pawn类型间**共享移动代码**。使用组件分隔不同功能是上佳方法，可在项目增大、Pawn越加复杂时减少杂乱。

从**PawnMovementComponent**派生出自定义的**MyPawnMovementComponent**（C++类）

与Actor类似，在TickComponent（）中定义逐帧移动方式

例子：

```c++
void UCollidingPawnMovementComponent::TickComponent(float DeltaTime, enum ELevelTick TickType, FActorComponentTickFunction *ThisTickFunction)     {         Super::TickComponent(DeltaTime, TickType, ThisTickFunction);          // 确保所有事物持续有效，以便进行移动。         
if (!PawnOwner || !UpdatedComponent || ShouldSkipUpdate(DeltaTime))         {             return;         }          
// 获取（然后清除）ACollidingPawn::Tick中设置的移动向量         
FVector DesiredMovementThisFrame = ConsumeInputVector().GetClampedToMaxSize(1.0f) * DeltaTime * 150.0f;         if (!DesiredMovementThisFrame.IsNearlyZero())         {             FHitResult Hit;             SafeMoveUpdatedComponent(DesiredMovementThisFrame, UpdatedComponent->GetComponentRotation(), true, Hit);             // 若发生碰撞，尝试滑过去 
if (Hit.IsValidBlockingHit())             {                 SlideAlongSurface(DesiredMovementThisFrame, 1.f - Hit.Time, Hit.Normal, Hit);             }         }     };
```

​	该代码将在场景中平滑移动Pawn，适当时在表面滑动。未对Pawn应用重力，其最大速度被硬编码为每秒150 **虚幻单位**。



此 `TickComponent` 函数使用 `UPawnMovementComponent` 类提供的几种强大功能。

- `ConsumeInputVector` 报告并清空用于存储移动输入的内置变量值。
- `SafeMoveUpdatedComponent` 利用虚幻引擎物理移动Pawn移动组件，同时考虑固体障碍。
- 移动导致碰撞时， `SlideAlongSurface` 会处理沿墙壁和斜坡等碰撞表面平滑滑动所涉及的计算和物理，而非直接停留原地，粘在墙壁或斜坡上。

Pawn移动组件中还包含众多值得探究的功能，但本教程范围中暂时无需使用。可查看如 **浮动Pawn移动**、**旁观者Pawn移动** 或 **角色移动组件**等其他类，了解额外使用范例和方法。

#### 同时使用Pawn和移动组件

首先仍然要将变量添加进MyPawn类进行追踪，仍添加到类定义

```c++
UPROPERTY()     
class UCollidingPawnMovementComponent* OurMovementComponent;
```

并在源文件中添加自定义移动组件的头文件

创建Pawn移动组件并将其与Pawn关联的方法非常简单。在 MyPawn构造函数底部，添加此代码：

```c++
// 创建移动组件的实例，并要求其更新根。     
OurMovementComponent = CreateDefaultSubobject<UCollidingPawnMovementComponent>(TEXT("CustomMovementComponent"));     OurMovementComponent->UpdatedComponent = RootComponent;
```

##### 注意：

​	与目前为止所见其他 **组件** 不同，无需将此组件附加到自己的组件层级。这是因为其他组件均属于 **场景组件** 类型，本身需要物理位置。但 **运动控制器** 并非场景组件，不代表物**理对象**，因此物理位置上存在或者与另一组件物理连接的概念不适用于此类控制器。

​	Pawn拥有名为 `GetMovementComponent` 的函数，用于提供给引擎中**其他类**访问该Pawn当前所用Pawn移动组件的权限。需覆盖该函数，使其返回自定义Pawn移动组件。在 `CollidingPawn.h` 中的类定义中，需添加：

```cpp
virtual UPawnMovementComponent* GetMovementComponent() const override;//虚继承
```

而在 `CollidingPawn.cpp` 中，需按以下所示，添加覆盖函数的定义：

```c++
UPawnMovementComponent* ACollidingPawn::GetMovementComponent() const
     {
         return OurMovementComponent;
     }
```

##### 把移动输入实现到Pawn移动组件中

​	首先在 `CollidingPawn.h` 中的类定义内声明以下函数：

```c++
void MoveForward(float AxisValue);
void MoveRight(float AxisValue);
void Turn(float AxisValue);
void ParticleToggle();
```

​	在 `CollidingPawn.cpp` 中，按以下所示，添加此类函数的定义：

```c++
void ACollidingPawn::MoveForward(float AxisValue)
         {
             if (OurMovementComponent && (OurMovementComponent->UpdatedComponent == RootComponent))
             {
                 OurMovementComponent->AddInputVector(GetActorForwardVector() * AxisValue);
             }
         }

         void ACollidingPawn::MoveRight(float AxisValue)
         {
             if (OurMovementComponent && (OurMovementComponent->UpdatedComponent == RootComponent))
             {
                 OurMovementComponent->AddInputVector(GetActorRightVector() * AxisValue);
             }
         }

         void ACollidingPawn::Turn(float AxisValue)
         {
             FRotator NewRotation = GetActorRotation();
             NewRotation.Yaw += AxisValue;
             SetActorRotation(NewRotation);
         }

         void ACollidingPawn::ParticleToggle()
         {
             if (OurParticleSystem && OurParticleSystem->Template)
             {
                 OurParticleSystem->ToggleActive();
             }
         }
```

​	现在只需将函数**绑定到输入事件**。将下列代码添加到 `ACollidingPawn::SetupPlayerInputComponent`：

```c++
InInputComponent->BindAction("ParticleToggle", IE_Pressed, this, &ACollidingPawn::ParticleToggle);

InInputComponent->BindAxis("MoveForward", this, &ACollidingPawn::MoveForward);

InInputComponent->BindAxis("MoveRight", this, &ACollidingPawn::MoveRight);

InInputComponent->BindAxis("Turn", this, &ACollidingPawn::Turn);
```



## 正式进入虚幻C++

