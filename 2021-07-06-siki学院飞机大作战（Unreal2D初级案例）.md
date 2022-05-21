---
title: siki学院飞机大作战（Unreal2D初级案例）
description: C++项目
#date: 2020-01-01 
updated: 2020-07-09
categories:
- UE4
tags:
- UE4
---

# 102-资源导入环境搭建

导入资源

项目设置改 Default Maps

Floor大小改成20倍

拖个Cube，y大小改成52.25等等，做四个墙

# 103-主角飞机创建

新建C++类，父类是Pawn，命名SpaceShip，选中Public

头文件里（up主建议用前置声明，节约时间。Google的建议参考[(1 封私信) 如何看待C++前置声明？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/63201378/answer/208096403)，我就没有用）

（更新，今天看了虚幻官方文档里的代码规范，建议用前置声明，所以下面代码里 include 的头文件应该去掉放到源文件里去）

```cpp
#include "Components/StaticMeshComponent.h"
#include "Components/SphereComponent.h"
// 头文件一定要加载#include "*.generated.h"这个类头文件之前，不然就报错。

// ...

protected:
	UPROPERTY(VisibleAnywhere, Category="Component")
	USphereComponent *CollisionComponent;

	UPROPERTY(VisibleAnywhere, Category="Component")
	UStaticMeshComponent *ShipSM;
	
```

源文件里构造函数

```cpp

// Sets default values
ASpaceShip::ASpaceShip()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	my_ship_static_mesh_component = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("my_ship_static_mesh_component"));
	RootComponent = my_ship_static_mesh_component;	// 网格体设置为根组件（值得一提的是，在后面107里根组件换成了碰撞体，因为需要调整网格体的朝向，而根组件不能旋转）

	my_collision_component = CreateDefaultSubobject<USphereComponent>(TEXT("my_collision_component"));
	my_collision_component->SetupAttachment(RootComponent);
}
```

# 104-控制飞机显示

编译后转为蓝图类，命名 BP_SpaceShip（UE5的编译按钮在右下角）

点进去，左边可以看到组件的层级关系。



先点 My Ship Static Mesh Component，Details 里 STATIC MESH 选择 SM_Ship_01。因为我们不用它进行碰撞检测，所以还有下面的 COLLISION 里的 Collision Presets，改成 NoCollision



创建一个蓝图类GameMode，命名BP_MyGameMode, 项目设置里把 Default GameMode 改成它。

然后把它的 DefaultPawnClass 改成 BP_SpaceShip，这个在刚刚项目设置的地方或者点进 BP_MyGameMode 都能改，效果是一样的。



创建一个相机组件附在根组件上，和上一节类似

头文件里

```cpp
	UPROPERTY(VisibleAnywhere, Category="Component")
	UCameraComponent *my_camera_component;
```

源文件里构造函数

```cpp
	my_camera_component = CreateDefaultSubobject<UCameraComponent>(TEXT("my_camera_component"));
	my_camera_component->SetupAttachment(RootComponent);
```



编译后在蓝图类里就能看到相机了，修改位置、旋转。

将投影模式改成 orthographic，这样就没有了远近的效果。调整 Ortho Width。



Player Start 是主角出生的位置

# 105-控制飞机朝向鼠标

这节课写了一个 `LookAtCursor()`函数，来控制飞机朝向鼠标，需要先获得 PlayerController 并且让它的鼠标位置显示出来

头文件里声明

```cpp

	void LookAtCursor();

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

```

源文件里实现

```cpp
// Called when the game starts or when spawned
void ASpaceShip::BeginPlay()
{
	Super::BeginPlay();

	my_player_controller_ = Cast<APlayerController>(GetController());
	my_player_controller_->bShowMouseCursor = true;
	
}

void ASpaceShip::LookAtCursor()
{
	FVector mouse_location, mouse_direction;
	my_player_controller_->DeprojectMousePositionToWorld(mouse_location, mouse_direction);	// 鼠标的 2D 位置转为 3D 位置和方向
	FVector target_location = FVector(mouse_location.X, mouse_location.Y, GetActorLocation().Z);
	
	FRotator rotator = UKismetMathLibrary::FindLookAtRotation(GetActorLocation(), target_location);	// 玩家看向鼠标需要旋转的角度
	SetActorRotation(rotator);
}

// Called every frame
void ASpaceShip::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	
	LookAtCursor();
}

```

这样就实现了飞机朝向鼠标，但是相机是跟随飞机的，下节课需要禁止相机跟随旋转

# 106-禁止摄像机跟随旋转

使用弹簧，比如赛车游戏拐弯相机延迟跟随。

在蓝图中将相机位置归零，就能看到弹簧臂绑定了相机

但还是会跟随旋转，我们需要在弹簧臂的 Details -> CAMERA SETTINGS 里把 Inherit Pitch/Yaw/Roll 的勾取消掉。

最后发现飞机的朝向不对。

# 107-更换根组件

飞机朝向不对，需要修改网格体的朝向，但是根组件无法旋转，将碰撞体设置为根组件，这样网格体才能旋转。

```cpp
// Sets default values
ASpaceShip::ASpaceShip()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	my_collision_component_ = CreateDefaultSubobject<USphereComponent>(TEXT("my_collision_component"));
	RootComponent = my_collision_component_;	// 把碰撞组件设为根组件
	
	my_ship_static_mesh_component_ = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("my_ship_static_mesh_component"));
	my_ship_static_mesh_component_->SetupAttachment(RootComponent);

	my_spring_arm_component = CreateDefaultSubobject<USpringArmComponent>(TEXT("my_spring_arm_component"));
	my_spring_arm_component->SetupAttachment(RootComponent);
	
	my_camera_component_ = CreateDefaultSubobject<UCameraComponent>(TEXT("my_camera_component"));
	my_camera_component_->SetupAttachment(my_spring_arm_component);
}

```

# 108-控制飞机移动

创建两个函数控制飞机的移动

```cpp

void ASpaceShip::MoveUP(float Value)
{
	// Value 是 1 或 -1
	// AddMovementInput(FVector(1, 0, 0), Value);
	AddMovementInput(FVector::UpVector, Value);	// 这种写法和上面是一样的，显得更高端些。（更新，下一节课把这里改成了 ForwardVector，因为 Up 是往 Z 轴方向）
	
}

void ASpaceShip::MoveRight(float Value)
{
	AddMovementInput(FVector::RightVector, Value);
}

// Called every frame
void ASpaceShip::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	
	LookAtCursor();
}

// Called to bind functionality to input
void ASpaceShip::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	// 这个函数第一个参数是轴名称，不一定要和绑定的函数名称一致
	PlayerInputComponent->BindAxis("MoveUp", this, &ASpaceShip::MoveUP);
	PlayerInputComponent->BindAxis("MoveRight", this, &ASpaceShip::MoveRight);

}


```

在项目设置的 Input 里添加轴映射，MoveUp 和 MoveRight，MoveUp 里加 W 和 S，MoveRight 里加 D 和 A

编译后并不能控制移动，因为我们主角继承的是 Pawn，而不是 Charecter

# 109-解决Pawn移动的问题

在头文件里添加一个变量表示速度，添加一个函数用来实现移动

```cpp
	UPROPERTY(EditAnywhere, Category="Move")
	float my_speed_;


	void Move(float TickDeltaTime);
```

源文件里这样实现

```cpp
void ASpaceShip::Move(float TickDeltaTime)
{
	// ConsumeMovementInputVector 函数获得用户输入的向量值
	// AddActorWorldOffset 函数基于世界坐标系运动，bSweep 设为 true 代表遇到障碍物停下
	AddActorWorldOffset(ConsumeMovementInputVector() * my_speed_ * TickDeltaTime, true);
}
```



还有一种方法是通过引入`/engine/Source/Runtime/Core/Public/Misc/App.h`获得`DeltaTime`，这样`Move`函数就不需要参数了

```cpp
#include "Misc/App.h"

void ASpaceShip::Move()
{
	// ConsumeMovementInputVector 函数获得用户输入的向量值
	// AddActorWorldOffset 函数基于世界坐标系运动，bSweep 设为 true 代表遇到障碍物停下
	AddActorWorldOffset(ConsumeMovementInputVector() * my_speed_ * FApp::GetDeltaTime(), true);
}

// Called every frame
void ASpaceShip::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	
	LookAtCursor();
	Move();
}
```

# 增强输入

我把输入改成了增强输入（Enhanced Input）

在蓝图里的内容浏览器创建一个 Input Mapping Context, 几个 Input Action，作为资产

在刚刚创建的 Input Mapping Context 里将 Input Action 和 按键绑定，并且加上合适的 Trigger 和 Modifier

头文件

```cpp
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"

protected:
	// 这三个选项会在蓝图里暴露出来，在蓝图里选择自己创建的输入资产
	UPROPERTY(EditDefaultsOnly, Category="EnhancedInput")
	UInputMappingContext *my_input_mapping_context_;
	
	UPROPERTY(EditDefaultsOnly, Category="EnhancedInput")
	UInputAction *my_move_up_;

	UPROPERTY(EditDefaultsOnly, Category="EnhancedInput")
	UInputAction *my_move_right_;
	

protected:
	// 注意，回调函数一定要加 UFUNCTION
	UFUNCTION()
	void MoveUp(const FInputActionValue& Value);

	UFUNCTION()
	void MoveRight(const FInputActionValue& Value);

```



源文件

```cpp
// Called when the game starts or when spawned
void ASpaceShip::BeginPlay()
{
	Super::BeginPlay();

	my_player_controller_ = Cast<APlayerController>(GetController());
	my_player_controller_->bShowMouseCursor = true;

	// 从玩家控制器关联的本地玩家获取增强输入本地玩家子系统系统（Enhanced Input Local Player Subsystem）。
	if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(my_player_controller_->GetLocalPlayer()))
	{
		// PawnClientRestart可以在Actor的生命周期内执行多次，所以记得先清空可能遗留的映射。
		Subsystem->ClearAllMappings();

		// 添加每个映射上下文以及它们的优先级数值。数值高意味着优先级更高。
		Subsystem->AddMappingContext(my_input_mapping_context_, 0);
	}
}


// Called to bind functionality to input
void ASpaceShip::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	// 这个函数第一个参数是轴名称，不一定要和绑定的函数名称一致
	// PlayerInputComponent->BindAxis("MoveUp", this, &ASpaceShip::MoveUP);
	// PlayerInputComponent->BindAxis("MoveRight", this, &ASpaceShip::MoveRight);

	// 请确保使用UEnhancedInputComponent；否则项目将无法正确配置。
	if (UEnhancedInputComponent* PlayerEnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		// 有多种方法可以将 UInputAction* 绑定给回调函数，并且有多个关联的 ETriggerEvent。

		// MyInputAction 开始后，此处会在每次tick时调用回调函数（例如按下某个操作按钮时）
		if (my_move_up_)
		{
			PlayerEnhancedInputComponent->BindAction(my_move_up_, ETriggerEvent::Triggered, this, &ASpaceShip::MoveUp);
		}

		//只要输入条件满足（例如按住某个方向键），此处会按名称在每次tick时调用回调函数（UFUNCTION）。
		if (my_move_right_)
		{
			PlayerEnhancedInputComponent->BindAction(my_move_right_, ETriggerEvent::Triggered, this, &ASpaceShip::MoveRight);
		}
	}
}

void ASpaceShip::MoveUp(const FInputActionValue& Value)
{
	// 输出调试日志，确保回调函数正常运行。
	// UE_LOG(LogTemp, Warning, TEXT("%s called with Input Action Value %s (magnitude %f)"), TEXT(__FUNCTION__), *Value.ToString(), Value.GetMagnitude());
	// 使用 GetType() 函数来确定Value的类型；请在运算符[]中使用0到2之间的索引值来访问数据。
	// 	// AddMovementInput(FVector(1, 0, 0), Value[0]);
		AddMovementInput(FVector::ForwardVector, Value[0]);	// 这种写法和上面是一样的，显得更高端些

}

void ASpaceShip::MoveRight(const FInputActionValue& Value)
{
	AddMovementInput(FVector::RightVector, Value[0]);
}

```



# 110-创建子弹

创建一个 C++ 子弹类，父类是 Actor，命名 Bullet，选中 Public

头文件里

```cpp

protected:

	UPROPERTY(VisibleAnywhere, Category="Component")
	USceneComponent *my_scene_component_;	// 是一个空组件，专门用来做根组件
	
	UPROPERTY(VisibleAnywhere, Category="Component")
	UStaticMeshComponent *my_bullet_static_mesh_component_;

```

源文件里

```cpp
#include "Components/SceneComponent.h"
#include "Components/StaticMeshComponent.h"
#include "Bullet.h"

// Sets default values
ABullet::ABullet()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	my_scene_component_ = CreateDefaultSubobject<USceneComponent>(TEXT("my_scene_component"));
	RootComponent = my_scene_component_;
	
	my_bullet_static_mesh_component_ = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("my_bullet_static_mesh_component"));
	my_bullet_static_mesh_component_->SetupAttachment(RootComponent);
	
}

```

将 C++ 类转换为蓝图类，命名 BP_Bullet，将网格体组件 Rotation Y 设为 -90 度

（有时候更新了代码，但是蓝图里没刷新出来，就在内容浏览器里 右键 -> Assert Actions -> Reload 就出来了）

# 111-子弹发射

需要在头文件里声明子弹和子弹发射的地方，还有发射的函数

```cpp
protected:
	UPROPERTY(VisibleAnywhere, Category="Component")
	USceneComponent *my_spawn_point_;	// 子弹从这里发射

	// TSubclassOf 模板类告知编辑器的属性窗口只列出派生自 ABullet 的类（作为属性选择）。
	UPROPERTY(EditAnywhere, Category = "Fire")
	TSubclassOf<ABullet> my_bullet_;

	UFUNCTION()
	void Fire();
```

源文件里，将子弹发射的地方绑到飞船的网格上，绑定开火Action，实现开火

```cpp
// Sets default values
ASpaceShip::ASpaceShip()
{
 	// Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	my_collision_component_ = CreateDefaultSubobject<USphereComponent>(TEXT("my_collision_component"));
	RootComponent = my_collision_component_;	// 把碰撞组件设为根组件
	
	my_ship_static_mesh_component_ = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("my_ship_static_mesh_component"));
	my_ship_static_mesh_component_->SetupAttachment(RootComponent);

	my_spring_arm_component = CreateDefaultSubobject<USpringArmComponent>(TEXT("my_spring_arm_component"));
	my_spring_arm_component->SetupAttachment(RootComponent);
	
	my_camera_component_ = CreateDefaultSubobject<UCameraComponent>(TEXT("my_camera_component"));
	my_camera_component_->SetupAttachment(my_spring_arm_component);

	my_spawn_point_ = CreateDefaultSubobject<USceneComponent>(TEXT("my_spawn_point"));
	my_spawn_point_->SetupAttachment(my_ship_static_mesh_component_);
	
	my_speed_ = 2500.f;
}

// Called to bind functionality to input
void ASpaceShip::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

    // 这几行废弃了，换成增强输入
	// 这个函数第一个参数是轴名称，不一定要和绑定的函数名称一致
	// PlayerInputComponent->BindAxis("MoveUp", this, &ASpaceShip::MoveUP);
	// PlayerInputComponent->BindAxis("MoveRight", this, &ASpaceShip::MoveRight);

	// 请确保使用UEnhancedInputComponent；否则项目将无法正确配置。
	if (UEnhancedInputComponent* PlayerEnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		// 有多种方法可以将 UInputAction* 绑定给回调函数，并且有多个关联的 ETriggerEvent。

		// MyInputAction 开始后，此处会在每次tick时调用回调函数（例如按下某个操作按钮时）
		if (my_move_up_)
		{
			PlayerEnhancedInputComponent->BindAction(my_move_up_, ETriggerEvent::Triggered, this, &ASpaceShip::MoveUp);
		}

		//只要输入条件满足（例如按住某个方向键），此处会按名称在每次tick时调用回调函数（UFUNCTION）。
		if (my_move_right_)
		{
			PlayerEnhancedInputComponent->BindAction(my_move_right_, ETriggerEvent::Triggered, this, &ASpaceShip::MoveRight);
		}

		if (my_fire_)
		{
			PlayerEnhancedInputComponent->BindAction(my_fire_, ETriggerEvent::Triggered, this, &ASpaceShip::Fire);
		}
	}
}



void ASpaceShip::Fire()
{
	// 实例化物体, 
	GetWorld()->SpawnActor<ABullet>(my_bullet_, my_spawn_point_->GetComponentLocation(), my_spawn_point_->GetComponentRotation());
}

```

蓝图里的内容浏览器创建开火的 Input Action，作为资产

在 Input Mapping Context 资产里将开火的 Input Action 和 按键绑定，并且加上合适的 Trigger 为 Pressed

在飞船蓝图里的 Fire 下的 My_Bullet 选择 BP_Bullet

在蓝图里调整一下 my_spawn_point_ 的位置和朝向，让它不要和飞船碰撞体重合



这样就可以点击鼠标左键发射了，子弹发射后会停留在原地

# 112-让子弹飞起来

这节课内容很少，因为用 UE 自带的组件就可以了

在子弹的代码里加上运动组件

头文件

```cpp
	// 上面还要前置声明一下
	UPROPERTY(VisibleAnywhere, Category="Component")
	UProjectileMovementComponent *my_projectile_movement_component_;
```

源文件

```cpp
	my_projectile_movement_component_ = CreateDefaultSubobject<UProjectileMovementComponent>(TEXT("my_projectile_movement_component"));	// 运动组件是不绑到根组件上的，而是并列的

```

编译后就可以在子弹蓝图里看到运动组件了，在它的 Detailes -> PROJECTILE 里修改 Initial Speed 为 3000，Projectile Gravity Scale 为 0（因为我们是 2D 游戏，不希望子弹往下掉）。

子弹有点小，在网格体组件里把缩放改成 1.5

# 113-连续开火

改成按住鼠标时，每 0.2 秒发射一颗子弹。用定时器实现

将原来的 Fire Action 删掉。

总结一下步骤，创建 Input Action 并使用需要的步骤

1. 创建两个 Input Action，Start Fire 和 End Fire

2. 在 Input Mapping Context 资产里将这两个 Input Action 与鼠标左键绑定，添加 Trigger Pressed 和 Released（UE5 的 Released 有 Bug，改成鼠标右键吧）

3. 在飞船 C++ 头文件里添加两个 Input Action 指针，暴露给蓝图

4. 在源文件里将这两个 Input Action 的 ETriggerEvent 与回调函数绑定

   ```cpp
   // Called to bind functionality to input
   void ASpaceShip::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
   {
   	Super::SetupPlayerInputComponent(PlayerInputComponent);
   
   	// 这几行废弃了，换成增强输入
   	// 这个函数第一个参数是轴名称，不一定要和绑定的函数名称一致
   	// PlayerInputComponent->BindAxis("MoveUp", this, &ASpaceShip::MoveUP);
   	// PlayerInputComponent->BindAxis("MoveRight", this, &ASpaceShip::MoveRight);
   
   	// 请确保使用UEnhancedInputComponent；否则项目将无法正确配置。
   	if (UEnhancedInputComponent* PlayerEnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
   	{
   		// 有多种方法可以将 UInputAction* 绑定给回调函数，并且有多个关联的 ETriggerEvent。
   
   		// MyInputAction 开始后，此处会在每次tick时调用回调函数（例如按下某个操作按钮时）
   		if (my_move_up_)
   		{
   			PlayerEnhancedInputComponent->BindAction(my_move_up_, ETriggerEvent::Triggered, this, &ASpaceShip::MoveUp);
   		}
   
   		//只要输入条件满足（例如按住某个方向键），此处会按名称在每次tick时调用回调函数（UFUNCTION）。
   		if (my_move_right_)
   		{
   			PlayerEnhancedInputComponent->BindAction(my_move_right_, ETriggerEvent::Triggered, this, &ASpaceShip::MoveRight);
   		}
   
   		if (my_start_fire_)
   		{
   			PlayerEnhancedInputComponent->BindAction(my_start_fire_, ETriggerEvent::Triggered, this, &ASpaceShip::StartFire);
   		}
   		
   		if (my_end_fire_)
   		{
   			PlayerEnhancedInputComponent->BindAction(my_end_fire_, ETriggerEvent::Triggered, this, &ASpaceShip::EndFire);
   		}
   	}
   }
   ```

5. 在源文件里实现回调函数，这里学会了定时器的用法

   要先在头文件里添加一个 `FTimerHandle my_timer_handle_between_shot_;`

   ```cpp
   #include "TimerManager.h"
   
   void ASpaceShip::StartFire()
   {
   	// UE_LOG(LogTemp, Warning, TEXT("start1"));
   	GetWorldTimerManager().SetTimer(my_timer_handle_between_shot_, this, &ASpaceShip::Fire, my_time_between_shot_, true, 0.0f);
   	// UE_LOG(LogTemp, Warning, TEXT("start2"));
   }
   
   void ASpaceShip::EndFire()
   {
   	// UE_LOG(LogTemp, Warning, TEXT("end1"));
   	GetWorldTimerManager().ClearTimer(my_timer_handle_between_shot_);
   	// UE_LOG(LogTemp, Warning, TEXT("end2"));
   }
   ```

6. 在蓝图类里为两个 Input Action 指针选择资产

# 114-敌人创建

这节课比较简单，创建 C++ 类，添加 Component，设置根组件，附加到根组件，转为蓝图类，选择网格体。

和创建自己的飞船的步骤很相似，就不再重复了。

# 115-敌人追逐主角移动

这节课比较简单，就是每帧改变敌人的位置和方向

头文件里创建能指向主角飞船的指针，源文件里在 BeginPlay 里讲指针指向主角。

然后完成 MoveTowardsPlayer 函数

```cpp
#include "Enemy.h"

#include "Components/StaticMeshComponent.h"
#include "Components/SphereComponent.h"
#include "Kismet/GameplayStatics.h"
#include "Misc/App.h"
#include "Kismet/KismetMathLibrary.h"

// Called when the game starts or when spawned
void AEnemy::BeginPlay()
{
	Super::BeginPlay();
	my_space_ship_ = Cast<ASpaceShip>(UGameplayStatics::GetPlayerPawn(this, 0));
}

void AEnemy::MoveTowardsPlayer()
{
	// 方向向主角
	FVector direction = (my_space_ship_->GetActorLocation() - GetActorLocation()).GetSafeNormal();
	// 移动
	AddActorWorldOffset(direction * my_speed_ * FApp::GetDeltaTime(), true);

	// 旋转朝向主角
	SetActorRotation(UKismetMathLibrary::FindLookAtRotation(GetActorLocation(), my_space_ship_->GetActorLocation()));
}

// Called every frame
void AEnemy::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	MoveTowardsPlayer();
}

```

# 116-敌人颜色随机生成

让敌人机身 4 种颜色，机翼 4 种颜色，一共 16 种

点击敌人蓝图里网格体组件的 MATERIALS 里的内容，可以看到机身是 Color1，机翼是 Color2，我们要在代码里给它们随机值。

添加一个设置颜色的函数，在蓝图里实现。

```cpp
protected:
	UFUNCTION(BlueprintImplementableEvent)
	void SetColor();
```

在蓝图里添加两个整数，分别代表机身的颜色 0-3，机翼的颜色 0-3。Set Color 先给这两个整数随机值

![image-20210726230922288](../img/210726/210726-SetColor1.png)

基于材质创建实例



双击线可以分叉



修改代码，让网格体在蓝图里可读
