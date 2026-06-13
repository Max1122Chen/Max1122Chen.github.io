# 虚幻5GAS(GameplayAbilitySystem)

核心：AbilitySystemComponent,简称ASC



### 深入GAS架构设计

#### GAS的优势

- 虽然很多情况下在角色中编写逻辑和用GAS编写逻辑达成的效果是一样的，但使用GAS为角色的技能定义冷却时间等更加方便
- 灵活易拓展，可实现复杂的技能流程
- 支持联机复制和预测回滚
- 易于团队协作和项目复用
- 细粒度思考实现单个动作逻辑
- 数据驱动数值配置
- 已帮你处理繁杂流程麻烦逻辑，只需专注在技能逻辑上



#### GAS的劣势

- 大量的概念和类，学习曲线陡峭
- 重度C++，对BP不友好
- 需要按照框架定义一堆类才能开始启动
- GAS的网络服务必须配合DS服务器框架
- 实践演化框架，可选插件，文档不足
- 要求技术功底高，源码Debug能力强



### 引入：如何设计一个易于扩展的技能系统

逻辑部分：

- 技能的获得和释放、效果
- 触发判断的条件
- Buff系统

视听部分：

- 动作动画
- 特效
- 声效

数据部分：

- 数值计算
- 数据配置

### GAS核心概念

- UAbilitySystemComponent——ASC

​	只有拥有ASC的Actor才具备获取、执行技能的能力

- UGameplayAbility——GA

​	定义技能逻辑

- UGameplayEffect——GE

​	定义一个技能的效果，修改属性或动作触发等

- UGameplayCueNotify——GC

​	定义特效部分

- FGameplayAttribute——Attribute

​	描述游戏属性，多个Attribute可以构成AttributeSet，挂载在Actor上

- FGameplayTag——Tag

​	受到广泛使用，通过给类和对象挂上标签可以层次化地判断和搜索各个条件

- UGameplayTask——Task

​	异步操作的节点，例子：在一个技能里播放一个蒙太奇，等蒙太奇结束后再结束这个技能

- FGameplayEventData——Event

​	通信

#### 总结：一个技能（GameplayAbility）的修养

- Who：谁能放技能？AbilitySystemComponent
- How：技能的逻辑？GameplayAbility
- Change：技能的效果？GameplayEffect
- What：技能的改变属性？GameplayAttribute
- If：技能涉及的条件？GameplayTag
- Visual：技能的视效？GameplayCue
- Async：技能的长时行动？GameplayTask
- Send：技能消息事件？GameplayEvent

### GAS核心结构

GameplayTag,GameplayAbility,GameplayTasks三个模块独立，1，3两者可脱离GAS存在

#### GameplayTags游戏标签

在ProjectSettings中可以找到编辑器中的所有GameplayTags

GameplayTag有轻量化，灵活的优势

#### GameplayAttribute游戏属性

BaseValue：基础值

CurrentValue：当前值，临时值

#### GameplayEffect技能效果

决定一个技能的”逻辑效果"

纯配置蓝图

### 使用GAS设计技能与属性

​	在定义技能的逻辑过程中，gameplay技能中的逻辑通常会调用一系列被称为 **技能任务** 异步编译块。技能任务衍生自抽象 `UAbilityTask` 类，以C++编写。其完成操作时，会基于最终结果频繁调用委托（C++中）或输出执行引脚（蓝图中）（例如，需要目标的技能需进行"瞄准"任务，将调用一个委托或输出引脚确认目标，并调用另一引脚取消技能）。

​	GameplayAttribute是存储在 `FGameplayAttribute` 结构中的"浮点"值，将对游戏或Actor产生影响；其通常为生命值、体力、跳跃高度、攻击速度等值,结构中包括两个值：该属性的CurrentValue和BaseValue。

​	**Gameplay效果（GE）**可即时或随时间改变Gameplay属性（通常称为"增益和减益"）。例如，施魔法时减少魔法值，激活"冲刺"技能后提升移动速度，或在治疗药物的效力周期内逐渐恢复生命值。



​	我们需要定义一个Game Ability基类，从这个基类再派生可使用的技能

​	要让角色获得技能，就需要让角色先“学会”技能，这需要在角色蓝图使用“Give Ability”

​	在角色蓝图中定义“激活技能”的方法时，由于一个角色可以有很多个技能，所以可以把“激活技能”封装成一个通用的函数，把角色的技能的标签存在一个数组中，通过给出索引指向标签数组，进而找到要使用的技能。

#### tag的使用

![image-20241026132543427](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026132543427.png)![image-20241026132636724](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026132636724.png)

为技能赋予一个Tag只需在技能的Tag->AbilityTag栏目下新增一个名字合适的标签，用"."，分隔层级即可。

1.tag可以标记一个技能，起分类的作用，使角色可以按类别调用一个技能

2.标记可以动态改变一个技能的“状态”，比如在技能应该进CD的时候，给技能打上一个阻止再次释放的tag，就可以保证技能在CD期间无法再次释放。



#### 用给技能定义CD的例子认识GE

在一个单独的技能GamePlayAbility中，Details下有一个Cooldown条目，可以为其配置在CD期间的GamePlayEffect![image-20241026133313153](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026133313153.png)![image-20241026133735289](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026133735289.png)



##### GameplayEffect：

只在GE中存数据，不要写逻辑



给CoolDown下配置一个GE，GE中配置一个阻止某标签的技能的使用，并给其赋上对应的技能。

**谨记要在激活技能后有必要的地方提交CD计时，告诉系统什么时候应该要开始计算CD，这可以区分CD计算不同的情况，例如使用技能技能立刻进CD、吟唱完成后技能进CD**



#### 属性设置

属性的设置需要用到C++，创建一个C++类，继承自AttributeSet类。

定义公有的FGameplayAttributeData类型的成员变量若干，它们就是角色属性。且我们还需要定义这些属性的最大值。

**首先在引入头文件：“AbilitySystemComponent.h"**

**范例如此：**![image-20241026140132416](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026140132416.png)

使用宏函数方便地帮助我们定义我们自定义的属性的get，set，init等方法

![image-20241026141615848](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026141615848.png)

完成的情况

![image-20241026141650863](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026141650863.png)

#### 属性值初始化

​	在UE中建立一个“数据表格”（其他下），选择AttributeMetaData。

​	数据表的命名要很讲究，要用属性类.成员变量。如BaseAttributeSet.HP

​	建好表后，把表设置到角色基类的AbilitySystem组件的Attibute Test中，并在属性中选择“BaseAttributeSet”



#### 设置普攻伤害

​	普攻伤害也是一种GE。在普攻的GE中设置“修饰符（Modified）”，设置作用属性为HP，作用效果是HP-5。

值得注意的是，此处的栏目名为“可扩展浮点幅度”，在后续我们可以为技能引入与等级相关的配表（CurveTable），此处的“可扩展浮点幅度”实际上是一个乘数。

![image-20241026144306847](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026144306847.png)

##### 普攻伤害溢出（属性值变成负值）的处理：

使用 后处理 对属性值做夹值处理，需要在头文件加上"GameplayEffectExtension.h"

![image-20241026154448425](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026154448425.png)

源文件中的写法如下：

![image-20241026154710519](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026154710519.png)

**完成！**



#### 题外话：镜头碰撞和延迟跟随

在SpringArm组件中不启用这一项可以实现不检测碰撞，但这一项可能会造成很多视角穿透的问题

![image-20241026155446707](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026155446707.png)

在SpringArm组件中启用这一项可以实现延迟跟随

![image-20241026155241360](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026155241360.png)



#### 回归正题——不同等级的伤害配表

在外部建立一个csv（要UTF-8的编码格式）格式的表，第一行写等级，第二行写对应等级的攻击力，再把它拽进UE中，引入成CurveTable，并选Constant.再在GEDamage中应用这个曲线表，就能根据不同等级造成不同伤害了。



### 生命值改变广播

​	在C++角色基类（BaseCharacter.h）下申请一个多播代理（声明一个动态多播委托，带一个参数，使用“动态多播”是为了在蓝图中能使用），并需要**添加头文件“GameplayEffectTypes.h”**

![image-20241026172100790](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026172100790.png)

​	声明一个委托变量，用于储存方法引用

![image-20241026172030194](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241026172030194.png)

​	当角色执行BeginPlay()即角色诞生后，角色将把我们自定义的一个方法绑定到UE给我们提供的一个当某个具体属性值改变时将执行的一个委托

<img src="C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241107123543964.png" alt="image-20241107123543964" style="zoom:150%;" />

​	调用的方法的具体实现，其中参数部分的FOnAttributeChangeData来自上面一张图所说的虚幻内置属性值改变委托，Data是一个结构体，存有改变前和改变后的值。注意此函数并非虚函数，否则会报错。在头文件声明的时候不要带上ABaseCharacter::，不然糖丸了

![image-20241107124035586](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241107124035586.png)

Broadcast()是为了执行HPChangeEvent引用的事件，参数列表中的值将传递给委托引用的事件

C++中代码即如上。此处可能有些令人困惑：1.我们并未给HPChangeEvent类型委托绑定任何方法。其实这个行为我们将在蓝图中实现，我们在BaseCharacter的派生类中可以给HPChangeEvent绑定某个我们希望执行的方法。



#### 总结

为什么要使用多播委托？生命值委托广播的逻辑是什么？

A：首先，在角色初始化时（BeginPlay），我们把一个自定义事件OnHealthAttributeChanged绑定给了UE给用户提供的一个委托，这个委托会在一个指定的属性值变化时执行，请注意，我们所谈论的属性值永远不是定义在角色自己身上的，而是通过AbilitySystem组件上挂载（指定）一个AttributeSet类实现的。

​	然后，在OnHealthAttributeChanged中，我们执行HPChangeEvent委托实例引用的事件，OnHealthAttributeChanged只相当于一个便于执行委托的外壳，实际效果就是通过Broadcast执行委托。这样一来，我们便可以在HP变化时执行任何我们在蓝图中定义的希望执行的逻辑了！



### 虚幻C++委托入门

#### C#中委托的简介

![image-20241107112842516](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241107112842516.png)

![image-20241107113015970](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241107113015970.png)

某种程度上来说，委托类似于为一种特定类型（即参数列表相同）的函数创造了一个别名，用户可以通过这个别名类似地调用函数。



#### 虚幻的委托

虚幻的委托的声明需要使用宏，一方面让虚幻引知道这是一个委托，另一方面虚幻的抽象魔改规定了委托的声明方法。

![image-20241107113801112](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241107113801112.png)

```c++
UDELEGATE(BlueprintAuthorityOnly)   //说明符
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FInstigatedAnyDamageSignature, float, Damage, const UDamageType*, DamageType, AActor*, DamagedActor, AActor*, DamageCauser);//宏定义
```

用法示例详见虚幻官方文档，比较清晰：[Delegates and Lamba Functions in Unreal Engine | 虚幻引擎 5.5 文档 | Epic Developer Community | Epic Developer Community (epicgames.com)](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/delegates-and-lamba-functions-in-unreal-engine)

## GAS使用思路

### GameplayAttributeSet的初始化思路

​	GameplayAttributeSet的初始化其实也就是多个GameplayAttribute的初始化了，虚幻官方文档4.27的ARPG示例中选择的思路是通过一个GE初始化GameplayAttribute，该GE被命名为**GE_StatsBase**，它的应用时机是角色初始化时获取技能时。

> **“瞬间（Instant）”时长** 表示该效果仅永久性地应用一次。然后，对于每个主要统计数据，都将具有一个 **属性修饰符**（Modified），它会基于 **CurveTable** 覆盖该值。**StartingStats** 从一个 **CSV**（位于 **Abilities/DataTables** 中）中导入，其中，针对每个统计数据都存在一行，针对每个关卡都存在一列。在本示例中，它将关注 **DefaultMaxHealth** 行，以及 **CharacterLevel** 列。**GE_PlayerStats** 效果会从该泛型效果中进行继承，并将所有行更改为 **PlayerMaxHealth** 等。通过以这种方式使用曲线表，可以轻松针对整个游戏一次性调整各个属性，无需手动逐个修改效果。也可以在游戏之外设置脚本，以创建 **CSV** 或 **JSON** 文件（从外部数据源创建，例如，**Excel**），并按照需要将它们导入。

### 一个近战技能的实现思路

​	虚幻4.27ARPG示例项目中一个近战技能GameplayAbility的基本框架

![image-20241130145054434](C:\Users\Max1122Chen\AppData\Roaming\Typora\typora-user-images\image-20241130145054434.png)

> 当启动某项能力时，调用 **ActivateAbility**，并使用 **CommitAbility** 应用能力的 **消耗（Cost）**（通常是ARPG中的法力）和 **冷却效果（Cooldown）**。调用 **EndAbility** 通知系统该能力已执行完毕。另外两个节点专用于ARPG，通常每个游戏都会根据需要添加新的函数和节点。

​	Play Montage and Wait for Event节点的解析：

> ​	**PlayMontageandWaitForEvent** 是一个 **AbilityTask** 节点，对应于 **URPGAbilityTask_PlayMontageAndWaitForEvent**。AbilityTask节点是一种特殊对象，它定义一组静态函数，以创建任务（在本例中为PlayMontageAndWaitForEvent）、变量和用于执行任务的函数。另外，还有一组动态/蓝图委托，从任务中激活。每个输出执行引脚（紧邻顶层，始终立即激活）对应于其中一个委托，输出数据引脚与委托签名相匹配。此特定任务是 **UAbilityTask_PlayMontageAndWait** 和 **UAbilityTask_WaitGameplayEvent** 的组合，其中包含一些特定于游戏的调整和注释。你的游戏可能需要实现几个新的Gameplay任务，因此此任务是如何进行设置的范例。



> 此任务的工作原理是首先播放蒙太奇，然后监听从 **AbilitySystemComponent** 发出的Gameplay事件。如果发出的Gameplay事件与传入的 **EventTags** 匹配，将使用标记和有效负载激活 **EventReceived** 执行引脚，然后调用 **ApplyEffectContainer** 函数。当蒙太奇混合、中断或取消时，该能力结束。实际的Gameplay事件使用以下逻辑从 **WeaponActor** 蓝图发出：

![arpg_melee20_abilities_02](C:\Users\Max1122Chen\Downloads\arpg_melee20_abilities_02.png)

> 这在武器actor与角色重叠时触发。在触发后，将构建 **GameplayEventData** 有效负载并传入 **Target Actor + Instigator**。然后，它使用放置在蒙太奇中的 **Anim Notify** 状态设置的标记发送Gameplay事件。因此，当该事件被触发后，能力图将激活 **EventReceived** 执行引脚。**ApplyEffectContainer** 节点对应于 **URPGGameplayAbility::ApplyEffectContainer**，它应用一组Gameplay效果。**每个URPGGameplayAbility**都有到 **FRPGGameplayEffectContainer** 结构的标记贴图（是Map 什么“贴图”），其中包含目标信息和要应用的Gameplay效果列表。以下是 **GA_PlayerAxeMelee** 中贴图（Map）的示例：

ApplyEffectContainer应该是该项目中用户自定义的新函数，**FRPGGameplayEffectContainer** 结构也应该是自定义的。

![arpg_melee20_abilities_03](C:\Users\Max1122Chen\Downloads\arpg_melee20_abilities_03.png)





## GAS深入详解——源码拆解

### AbilitySystemComponent类



### Attribute类

前置头文件引入和类与结构体的事先声明

```c++
// Copyright Epic Games, Inc. All Rights Reserved.

#pragma once

#include "CoreMinimal.h"
#include "Templates/SubclassOf.h"
#include "UObject/UnrealType.h"
#include "Engine/DataTable.h"
#include "AttributeSet.generated.h"

class UAbilitySystemComponent;
class UAttributeSet;
class UCurveTable;
struct FGameplayAbilityActorInfo;
struct FAggregator;
```

​	下面的FGameAttributeData结构体塑造AttributeSet中的基本单位Attribute的数据形态，它包含两个float类型成员变量：CurrentValue和BaseValue。

```c++
/** Place in an AttributeSet to create an attribute that can be accesed using FGameplayAttribute. It is strongly encouraged to use this instead of raw float attributes */

/**放在一个AttributeSet中来创造一个可以通过FGameplayAttribute获取的逻辑上的Attribute。强烈建议用这个结构体而不是只是用一个单纯的float作为逻辑上的Attribute。*/
USTRUCT(BlueprintType)
struct GAMEPLAYABILITIES_API FGameplayAttributeData
{
	GENERATED_BODY()
	FGameplayAttributeData()
		: BaseValue(0.f)
		, CurrentValue(0.f)
	{}

	FGameplayAttributeData(float DefaultValue)
		: BaseValue(DefaultValue)
		, CurrentValue(DefaultValue)
	{}

	virtual ~FGameplayAttributeData()
	{}

	/** Returns the current value, which includes temporary buffs */
	float GetCurrentValue() const;

	/** Modifies current value, normally only called by ability system or during initialization */
	virtual void SetCurrentValue(float NewValue);

	/** Returns the base value which only includes permanent changes */
	float GetBaseValue() const;

	/** Modifies the permanent base value, normally only called by ability system or during initialization */
	virtual void SetBaseValue(float NewValue);

protected:
	UPROPERTY(BlueprintReadOnly, Category = "Attribute")
	float BaseValue;

	UPROPERTY(BlueprintReadOnly, Category = "Attribute")
	float CurrentValue;
};
```

没看懂下面在干嘛。

```c++
/** Describes a FGameplayAttributeData or float property inside an attribute set. Using this provides editor UI and helper functions */
USTRUCT(BlueprintType)
struct GAMEPLAYABILITIES_API FGameplayAttribute
{
	GENERATED_USTRUCT_BODY()

	FGameplayAttribute()
		: Attribute(nullptr)
		, AttributeOwner(nullptr)
	{
	}

	FGameplayAttribute(FProperty *NewProperty);

	bool IsValid() const
	{
		return Attribute != nullptr;
	}

	/** Set up from a FProperty inside a set */
	void SetUProperty(FProperty *NewProperty)
	{
		Attribute = NewProperty;
		if (NewProperty)
		{
			AttributeOwner = Attribute->GetOwnerStruct();
			Attribute->GetName(AttributeName);
		}
		else
		{
			AttributeOwner = nullptr;
			AttributeName.Empty();
		}
	}

	/** Returns raw property */
	FProperty* GetUProperty() const
	{
		return Attribute.Get();
	}

	/** Returns the AttributeSet subclass holding this attribute */
	UClass* GetAttributeSetClass() const
	{
		check(Attribute.Get());
		return CastChecked<UClass>(Attribute->GetOwner<UObject>());
	}

	/** Returns true if this is one of the special attributes defined on the AbilitySystemComponent itself */
	bool IsSystemAttribute() const;

	/** Returns true if the variable associated with Property is of type FGameplayAttributeData or one of its subclasses */
	static bool IsGameplayAttributeDataProperty(const FProperty* Property);

	/** Modifies the current value of an attribute, will not modify base value if that is supported */
	void SetNumericValueChecked(float& NewValue, class UAttributeSet* Dest) const;

	/** Returns the current value of an attribute */
	float GetNumericValue(const UAttributeSet* Src) const;
	float GetNumericValueChecked(const UAttributeSet* Src) const;

	/** Returns the AttributeData, will fail if this is a float attribute */
	const FGameplayAttributeData* GetGameplayAttributeData(const UAttributeSet* Src) const;
	const FGameplayAttributeData* GetGameplayAttributeDataChecked(const UAttributeSet* Src) const;
	FGameplayAttributeData* GetGameplayAttributeData(UAttributeSet* Src) const;
	FGameplayAttributeData* GetGameplayAttributeDataChecked(UAttributeSet* Src) const;
	
	/** Equality/Inequality operators */
	bool operator==(const FGameplayAttribute& Other) const;
	bool operator!=(const FGameplayAttribute& Other) const;

	friend uint32 GetTypeHash( const FGameplayAttribute& InAttribute )
	{
		// FIXME: Use ObjectID or something to get a better, less collision prone hash
		return PointerHash(InAttribute.Attribute.Get());
	}

	/** Returns name of attribute, usually the same as the property */
	FString GetName() const
	{
		return AttributeName.IsEmpty() ? *GetNameSafe(Attribute.Get()) : AttributeName;
	}

#if WITH_EDITORONLY_DATA
	/** Custom serialization */
	void PostSerialize(const FArchive& Ar);
#endif

	/** Name of the attribute, usually the same as property name */
	UPROPERTY(Category = GameplayAttribute, VisibleAnywhere, BlueprintReadOnly)
	FString AttributeName;

	/** In editor, this will filter out properties with meta tag "HideInDetailsView" or equal to FilterMetaStr. In non editor, it returns all properties */
	static void GetAllAttributeProperties(TArray<FProperty*>& OutProperties, FString FilterMetaStr=FString(), bool UseEditorOnlyData=true);

private:
	friend class FAttributePropertyDetails;

	UPROPERTY(Category = GameplayAttribute, EditAnywhere)
	TFieldPath<FProperty> Attribute;
	//FProperty*	Attribute;

	UPROPERTY(Category = GameplayAttribute, VisibleAnywhere)
	TObjectPtr<UStruct> AttributeOwner;
};
```

没看懂下面

```c++
#if WITH_EDITORONLY_DATA
template<>
struct TStructOpsTypeTraits< FGameplayAttribute > : public TStructOpsTypeTraitsBase2< FGameplayAttribute >
{
	enum
	{
		WithPostSerialize = true,
	};
};
#endif
```

AttributeSet基类的定义

```c++
/**
 * Defines the set of all GameplayAttributes for your game
 * Games should subclass this and add FGameplayAttributeData properties to represent attributes like health, damage, etc
 * AttributeSets are added to the actors as subobjects, and then registered with the AbilitySystemComponent
 * It often desired to have several sets per project that inherit from each other
 * You could make a base health set, then have a player set that inherits from it and adds more attributes
 */
UCLASS(DefaultToInstanced, Blueprintable)
class GAMEPLAYABILITIES_API UAttributeSet : public UObject
{
	GENERATED_UCLASS_BODY()

public:
	// Populates (without emptying) a TArray with all FGameplayAttributes from an attribute set class.
	static void GetAttributesFromSetClass(const TSubclassOf<UAttributeSet>& AttributeSetClass, TArray<FGameplayAttribute>& Attributes);

	/** Override to disable initialization for specific properties */
	virtual bool ShouldInitProperty(bool FirstInit, FProperty* PropertyToInit) const { return true; }

	/**
	 *	Called just before modifying the value of an attribute. AttributeSet can make additional modifications here. Return true to continue, or false to throw out the modification.
	 *	Note this is only called during an 'execute'. E.g., a modification to the 'base value' of an attribute. It is not called during an application of a GameplayEffect, such as a 5 ssecond +10 movement speed buff.
	 */	
	virtual bool PreGameplayEffectExecute(struct FGameplayEffectModCallbackData &Data) { return true; }
	
	/**
	 *	Called just after a GameplayEffect is executed to modify the base value of an attribute. No more changes can be made.
	 *	Note this is only called during an 'execute'. E.g., a modification to the 'base value' of an attribute. It is not called during an application of a GameplayEffect, such as a 5 ssecond +10 movement speed buff.
	 */
	virtual void PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData &Data) { }

	/**
	 *	An "On Aggregator Change" type of event could go here, and that could be called when active gameplay effects are added or removed to an attribute aggregator.
	 *	It is difficult to give all the information in these cases though - aggregators can change for many reasons: being added, being removed, being modified, having a modifier change, immunity, stacking rules, etc.
	 */

	/**
	 *	Called just before any modification happens to an attribute. This is lower level than PreAttributeModify/PostAttribute modify.
	 *	There is no additional context provided here since anything can trigger this. Executed effects, duration based effects, effects being removed, immunity being applied, stacking rules changing, etc.
	 *	This function is meant to enforce things like "Health = Clamp(Health, 0, MaxHealth)" and NOT things like "trigger this extra thing if damage is applied, etc".
	 *	
	 *	NewValue is a mutable reference so you are able to clamp the newly applied value as well.
	 */
	virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) { }

	/** Called just after any modification happens to an attribute. */
	virtual void PostAttributeChange(const FGameplayAttribute& Attribute, float OldValue, float NewValue) { }

	/**
	 *	This is called just before any modification happens to an attribute's base value when an attribute aggregator exists.
	 *	This function should enforce clamping (presuming you wish to clamp the base value along with the final value in PreAttributeChange)
	 *	This function should NOT invoke gameplay related events or callbacks. Do those in PreAttributeChange() which will be called prior to the
	 *	final value of the attribute actually changing.
	 */
	virtual void PreAttributeBaseChange(const FGameplayAttribute& Attribute, float& NewValue) const { }

	/** Called just after any modification happens to an attribute's base value when an attribute aggregator exists. */
	virtual void PostAttributeBaseChange(const FGameplayAttribute& Attribute, float OldValue, float NewValue) const { }

	/** Callback for when an FAggregator is created for an attribute in this set. Allows custom setup of FAggregator::EvaluationMetaData */
	virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const { }

	/** This signifies the attribute set can be ID'd by name over the network. */
	void SetNetAddressable();

	/** Initializes attribute data from a meta DataTable */
	virtual void InitFromMetaDataTable(const UDataTable* DataTable);

	/** Gets information about owning actor */
	AActor* GetOwningActor() const;
	UAbilitySystemComponent* GetOwningAbilitySystemComponent() const;
	UAbilitySystemComponent* GetOwningAbilitySystemComponentChecked() const;
	FGameplayAbilityActorInfo* GetActorInfo() const;

	/** Print debug information to the log */
	virtual void PrintDebug();

	// Overrides
	virtual bool IsNameStableForNetworking() const override;
	virtual bool IsSupportedForNetworking() const override;
	virtual void PreNetReceive() override;
	virtual void PostNetReceive() override;

#if UE_WITH_IRIS
	/** Register all replication fragments */
	virtual void RegisterReplicationFragments(UE::Net::FFragmentRegistrationContext& Context, UE::Net::EFragmentRegistrationFlags RegistrationFlags) override;
#endif // UE_WITH_IRIS
protected:
	/** Is this attribute set safe to ID over the network by name?  */
	uint32 bNetAddressable : 1;
};

```

​	FAttributeMetaData的定义，使得用户可以通过一个以AttributeMetaData为元数据的DataTable初始化一个AttributeSet，但还没开发完。

```c++
/**
 *	DataTable that allows us to define meta data about attributes. Still a work in progress.
 */
USTRUCT(BlueprintType)
struct GAMEPLAYABILITIES_API FAttributeMetaData : public FTableRowBase
{
	GENERATED_USTRUCT_BODY()

public:

	FAttributeMetaData();

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Gameplay Attribute")
	float		BaseValue;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Gameplay Attribute")
	float		MinValue;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Gameplay Attribute")
	float		MaxValue;

	UPROPERTY()
	FString		DerivedAttributeInfo;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Gameplay Attribute")
	bool		bCanStack;
};
```

​	下面给了一堆宏函数，他们可以方便地帮助创建FGameplayAttribute类型的结构体，而这些结构体对应着各自的FGameplayAttributeData结构体。在虚幻的该架构中，我们需要通过FGameplayAttribute类型才能使用一些预处理和后处理函数，因为他们接受的参数是FGameplayAttribute类型，所以，使用这种类型是必要的。

​	通过ATTRIBUTE_ACCESSORS(ClassName, PropertyName)宏函数，为我们的AttributeSet中的每一个Attribute赋予“完全的肉身”就非常方便了，只需给它你的AttributeSet的类名，和对应的属性名，它就能展开为Attribute塑造肉身，而我们只需定义一些FGameplayAttributeData就行了。

```c++
/**
 * This defines a set of helper functions for accessing and initializing attributes, to avoid having to manually write these functions.
 * It would creates the following functions, for attribute Health
 *
 *	static FGameplayAttribute UMyHealthSet::GetHealthAttribute();
 *	FORCEINLINE float UMyHealthSet::GetHealth() const;
 *	FORCEINLINE void UMyHealthSet::SetHealth(float NewVal);
 *	FORCEINLINE void UMyHealthSet::InitHealth(float NewVal);
 *
 * To use this in your game you can define something like this, and then add game-specific functions as necessary:
 * 
 *	#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
 *	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
 *	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
 *	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
 *	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
 * 
 *	ATTRIBUTE_ACCESSORS(UMyHealthSet, Health)
 */

#define GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	static FGameplayAttribute Get##PropertyName##Attribute() \
	{ \
		static FProperty* Prop = FindFieldChecked<FProperty>(ClassName::StaticClass(), GET_MEMBER_NAME_CHECKED(ClassName, PropertyName)); \
		return Prop; \
	}

#define GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	FORCEINLINE float Get##PropertyName() const \
	{ \
		return PropertyName.GetCurrentValue(); \
	}

#define GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	FORCEINLINE void Set##PropertyName(float NewVal) \
	{ \
		UAbilitySystemComponent* AbilityComp = GetOwningAbilitySystemComponent(); \
		if (ensure(AbilityComp)) \
		{ \
			AbilityComp->SetNumericAttributeBase(Get##PropertyName##Attribute(), NewVal); \
		}; \
	}

#define GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName) \
	FORCEINLINE void Init##PropertyName(float NewVal) \
	{ \
		PropertyName.SetBaseValue(NewVal); \
		PropertyName.SetCurrentValue(NewVal); \
	}
```

