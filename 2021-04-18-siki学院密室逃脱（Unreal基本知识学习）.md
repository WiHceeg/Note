---
title: 2021-04-18-siki学院密室逃脱（Unreal基本知识学习）
description: 基础操作
#date: 2018-11-22 
#updated: 2020-07-25
categories:
- UE4
tags:
- UE4
---

**这个项目做的是一个房间，有个门是关着的，玩家需要逃出去。**
**玩家跑到房间的一个亮灯的角落，门会打开，想出去门又会关上。**
**玩家需要把桌子和椅子都搬到角落，门打开了，才能逃出去。**




 按住左键或右键后滑滚轮，会改变wasdqe的灵敏度

------

Viewport单击后按F，或者World Ourline双击，可以聚焦到某个actor。
聚焦后按住Alt同时左键按住移动，可以旋转

------

同时按住鼠标左键右键，可以平移

------

WER控制平移、旋转、缩放

------

右上角snap to grid 对齐网格。还有角度、缩放的snap

------

左上角调显示模式，光照

------

怎样让物体正好放在地面上，按End键将物体落地，也可以掉在别的东西上面

------

拖动的时候按住Shift跟随

------

按住Alt拖动是复制

------

播放后按Esc停止，按Shift + F1从View离开 

------

双击Detial里的StaticMesh，菜单栏Collision -> Add Box Simple Collision，还有Collision -> Simple Collision

------

Details里Add component，New C++ component

------

C++类里的项目名+ProjectGameModeBase是游戏相关的设置

------

属性初始化(GetOwner得到所在的Actor)放在构造函数里，操作放在BeginPlay()里

------

在World Outliner里，可以把同类的放到新建文件夹里

------

UE4中	// pitch=y, yaw=x, roll=x

```cpp
//让门开局就是开的
FRotator NewRotator = FRotator(0, -90, 0);
Owner->SetActorRotation(NewRotator);
```

------

Lights -> Spot Light添加聚光灯，可以调节颜色，内外光角度等

------

Volumes -> Trigger Volume，添加一个触发的体积

------

	UPROPERTY(EditAnywhere)		//细节面板上就会出现，可以指定字段
	ATriggerVolume* TriggerVolume;
------

重新编译，细节面板OpenDoor里就会重新出现Open Door -> TriggerVolume, 可以选，也可以锁定再拖拽上面的下来

------

BeginPlay()游戏开始时调用，TickComponent()每一帧都调用

------

​	GENERATED_BODY()如果红字了，解决方案资源管理器里引用->添加引用，就好了

------

游戏开始后，右侧的World Outliner会出现一些黄色的，是游戏创建出来之后出现的。其中有一个DefaultPawn是主角，按F8可以让游戏暂停，这时候就可以看到主角是一个球

------

GetWorld()获得UWorld对象，从世界的范围内管理整个游戏

```cpp
DefaultPawn = GetWorld()->GetFirstPlayerController()->GetPawn();
```

------

TickComponent()里是每一帧都调用，这里面检测DefaultPawn是否到了TriggerVolume里

```cpp
	if (TriggerVolume->IsOverlappingActor(DefaultPawn)) {
		OpenDoor();
	}
	else {
		CloseDoor();
	}

```

------

上面的代码，最好先判断指针是否非空，这样可以防止崩溃

```cpp
	if (TriggerVolume != nullptr && TriggerVolume->IsOverlappingActor(DefaultPawn)) {
		OpenDoor();
	}
```

------

实现延时关门

```cpp
	if (TriggerVolume != nullptr && TriggerVolume->IsOverlappingActor(DefaultPawn)) {
		OpenDoor();
		LastDoorOpenTime = GetWorld()->GetTimeSeconds();
	}
	if(GetWorld()->GetTimeSeconds() - LastDoorOpenTime > DoorOpenDuration) {
		CloseDoor();
	}
```

------

初学者内容Props里有玻璃，可以做个屋顶

------

UG_LOG在日志中打印信息

```cpp
	UE_LOG(LogTemp, Warning, TEXT("Hello %s is in OpenDoor!"), *Owner->GetName());
```

------

主角不想使用默认创建的，将游戏开始后出现的Default在蓝图下拉转换为蓝图类

------

项目设置里Maps & Modes创建新游戏模式，使用这个新游戏模式就可以把Default Pawn Class改为自己指定的

------

在自己创建的主角里Add Component，添加一个C++ Component，Actor Component，命名为Grabber叫抓取器

------

希望在鼠标正前方1米可以按鼠标左键抓取，用射线检测

------

PlayerController是用来控制player移动的，现在Grabber的每一帧里画DebugLine

```cpp
	FVector Start;
	FRotator ViewRotator;

	GetWorld()->GetFirstPlayerController()->GetPlayerViewPoint(Start, ViewRotator);

	FVector End = Start + ViewRotator.Vector() * 100;
	
	DrawDebugLine(GetWorld(), Start, End, FColor(255, 0, 0), false, 0, 0, 10);

```

------

碰撞检测部分

```cpp
	FHitResult Hit;
	GetWorld()->LineTraceSingleByObjectType(
		Hit,
		Start,
		End,
		FCollisionObjectQueryParams(ECollisionChannel::ECC_PhysicsBody),
		FCollisionQueryParams(FName(TEXT("")), false, GetOwner())
	);
	AActor* Actor = Hit.GetActor();
	if (Actor != nullptr) {
		UE_LOG(LogTemp, Warning, TEXT("Line Hit:%s"), *Actor->GetName());
	}
```

------

还需要在物体的Details里勾选上Simulate Physics，才能碰撞

------

按键在，项目设置-> Engine-> Input -> Bindings -> Action Mapping，添加命名为Grab，添加左键和空格

------

运行后，主角会生成一个InputComponent，用来做事件监测，所以我们要先得到InputComponent，通过它来获取Action事件的触发

头文件中

```cpp
	UInputComponent* InputComponent = nullptr;
```

BeginPlay()中，用FindComponentByClass()通过class查找组件。BindAction绑定叫Grab的Action，把当前对象和要触发的函数绑定

```cpp
void UGrabber::BeginPlay()
{
	Super::BeginPlay();

	// ...
	InputComponent = GetOwner()->FindComponentByClass<UInputComponent>();
	if (InputComponent != nullptr) {
		UE_LOG(LogTemp, Warning, TEXT("Input Component is founded!"));
		InputComponent->BindAction("Grab", IE_Pressed, this, &UGrabber::Grab);
		InputComponent->BindAction("Grab", IE_Released, this, &UGrabber::Release);

	}
	else {
		UE_LOG(LogTemp, Error, TEXT("Input Component is not founded!"));
	}
}

```

------

通过PhysicsHandle控制物体的抓取

先在主角的蓝图里Add Component，添加PhysicsHandle

在头文件里

```cpp
	UPhysicsHandleComponent* PhysicsHandle = nullptr;
```
BeginPlay()中，获取组件

```cpp
	PhysicsHandle = GetOwner()->FindComponentByClass<UPhysicsHandleComponent>();
```

PhysicsHandle->GrabComponent()需要的参数不是物品，而是组件，所以LineTrace()要改成返回组件。

自己定义的Grab函数和Release函数这样写

```cpp
void UGrabber::Grab()
{
	UE_LOG(LogTemp, Warning, TEXT("Grab action is pressed!"));
	FHitResult Hit = LineTrace();
	UPrimitiveComponent* ComponentToGrab = Hit.GetComponent();

	FRotator NewRotator = FRotator(0, 0, 0);

	if (Hit.GetActor() != nullptr && PhysicsHandle != nullptr) {
		
		PhysicsHandle->GrabComponentAtLocationWithRotation(
			ComponentToGrab,
			NAME_None,
			ComponentToGrab->GetOwner()->GetActorLocation(),
			NewRotator
		);
	}
}

void UGrabber::Release()
{
	UE_LOG(LogTemp, Warning, TEXT("Grab action is released!"));
	if (PhysicsHandle != nullptr) {
		PhysicsHandle->ReleaseComponent();
	}
}
```

------

修改机关的触发方式，椅子和桌子都搬过去才开门，在Details里Physics修改重量，在Collision里勾选上Generate Overlap Events

改自身的物理效果，主角的Collision Component，在Details里勾上Simulate Physics，Constraints里Lock Rotation防止它

计算重量，在每一帧检测判断

```cpp
float UOpenDoor::GetTotalMassInTrigger()
{
	TArray<AActor*> Actors;

	if (TriggerVolume == nullptr) { return 0.0; }
	TriggerVolume->GetOverlappingActors(OUT Actors);
	
	float TotalMass = 0;
	for (const auto& Actor : Actors) {
		TotalMass += Actor->FindComponentByClass<UPrimitiveComponent>()->GetMass();
	}
	UE_LOG(LogTemp, Warning, TEXT("Total Mass is %f!"), TotalMass);

	return TotalMass;
}

void UOpenDoor::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	// ...

	//if (TriggerVolume != nullptr && TriggerVolume->IsOverlappingActor(DefaultPawn)) {
	if (TriggerVolume != nullptr && GetTotalMassInTrigger() > 25) {

		OpenDoor();
		LastDoorOpenTime = GetWorld()->GetTimeSeconds();

	}
	if(GetWorld()->GetTimeSeconds() - LastDoorOpenTime > DoorOpenDuration) {
		CloseDoor();
	}
}

```

------

添加动画，把门转换选中的Actor为蓝图类，再把Event Graph里的节点都删掉。将C++开门和关门给蓝图负责，定义事件，需要开门的时候触发，蓝图也能监听到

```cpp
//头文件里
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FDoorEvent);

UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class ROOMESCAPE_API UOpenDoor : public UActorComponent
{
	GENERATED_BODY()

public:	
	// Sets default values for this component's properties
	UOpenDoor();

	UPROPERTY(BlueprintAssignable)
	FDoorEvent OnOpen;
	UPROPERTY(BlueprintAssignable)
	FDoorEvent OnClose;
```

```cpp
void UOpenDoor::OpenDoor()
{
	// pitch=y, yaw=x, roll=x
	FRotator NewRotator = FRotator(0, -90, 0);
	Owner->SetActorRotation(NewRotator);
	OnOpen.Broadcast();
}

```

在门的蓝图里OpenDoor右键Add Event，OnOpen，先从箭头拖出Print String测试下

------

测试成功，我们就不需要OpenDoor()和CloseDoor()这两个函数了，也不要延时，只要在TickComponent里触发OnOpen和OnClose的广播

```cpp
void UOpenDoor::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	// ...

	if (TriggerVolume != nullptr && GetTotalMassInTrigger() > 25) {

		OnOpen.Broadcast();

	}
	else {
		OnClose.Broadcast();
	}
```

------

在门的蓝图里OnOpen拖出Timeline，双击添加，选浮点型的时间轴，让它从0秒0到1秒90

------

给主角碰撞里的Physics里的旋转约束锁定，主角不控制的时候就不会滚了

------

导入wav，蓝图里关键帧Play Music

------

