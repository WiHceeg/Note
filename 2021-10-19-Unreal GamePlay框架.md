---
title: GamePlay框架
description: 
#date: 2018-11-22 
#updated: 2020-07-25
categories:
- Unreal
tags:
- Unreal 
---

# GameInstance（游戏实例）
贯穿一个游戏始终的东西，不关关卡是否在进行，是否在等待界面，是否已经结束了，都是可以访问到GameInstance的，适合放置一些独立于关卡的信息，以及一些功能，例如，显示UI登录，UI独立于任何关卡，比如刚打开一个游戏进入主界面/登录界面，就可以把显示它的逻辑放在GameInstance里。

---

游戏一旦开始便产生了一个游戏实例（GameInstance）

他的生命随着游戏程序的开始而建立，随着游戏程序的结束而消亡。

用途：

1.在GameInstance中保存需要初始化的全局变量方便后续应用（全局变量使用请慎重，生命周期长占内存，会造成全局污染，强耦合，且不易管理）

2.编写关卡跳转逻辑

# GameMode （游戏模式）
指的是游戏玩法规则，里面通常会写明游戏玩法，游戏进行的规则，胜负条件等等，游戏模式是和关卡绑定的，一个关卡只能有一个游戏模式，但是一整个游戏可以有多少GameMode。比如Moba游戏的LOL有不同的游戏模式，5V5模式拆掉水晶胜利，还有轮换模式比拼杀敌数胜利的模式，都属于不同的GameMode。

# GameState（游戏状态）
游戏状态，通常用于记录游戏模式规定好规则之后，GameState用来记录因为GameMode所产生的信息和数据，GameMode通过和GameState绑定使用的，一个是抽象的规则，一个是因为规则产生的具体信息和数据

# Pawn（可以被控制对象）
可以是玩家，也可以是敌人，也可以是NPC，LOL中玩家控制的英雄，敌人野怪，都是Pawn，Pawn不一定人，也不一定是活物，比如在球球大作战中，球是被玩家控制的对象，也是Pawn。框架中Pawn代表了一切具有运动形态的物体，即拥有运动形态的物体应该继承于该类。

# Character（可以被控制的人形对象）
继承自Pawn的一种特殊Pawn，但设计为类人的，并附带有人的基础属性和特征动作，大多情况下游戏的主角(即玩家)应该继承于该类。

# PlayerState（玩家状态）
玩家的信息和数据一般就记录在PlayerState里，也就是说玩家如果是Pawn/ Character，那玩家的信息和数据就记录在PlayerState里面，有点类似于之前GameMode和GameState的关系一样，是绑定使用的，GameState和PlayerState就像两个信息收集员，负责收集游戏中主要的信息和数据。

# Controller（控制器）
Controller拥有一个Pawn类实例，负责控制该Pawn应有的行为。把Pawn比作木偶，Controller就是控制这个木偶的控制器，一个Controller可以持有一个Pawn，一般玩家死亡的时候先会销毁Controller持有的Pawn，然后在创建一个新的Pawn让Controller持有。通常通过Controller作为媒介来操控角色。也可以把一些不要同Pawn同时销毁的信息存放在Controller中。

# PlayController（玩家控制器）
玩家控制器，属于游戏玩家的控制器，继承自Controller（控制器，本质上代表了人类玩家的意愿。
官方对PlayController的描述是：您可以在 Pawn 中处理所有输入， 尤其是不太复杂的情况下。但是，如果您的需求非常复杂，比如在一个游戏客户端上的多玩家、或实时地动态修改角色的功能，那么最好 PlayerController中处理输入。在这种情况中，PlayerController决定要干什么，然后将命令（比如"开始蹲伏"、“跳跃”）发布给Pawn。

# AIController（AI控制器）
AIController继承于Controller，拥有一个Pawn的实例，下AIController继承于Controller，拥有一个Pawn的实例，主要负责控制非玩家控制的有运动形态的物体，比如AI等等。

# HUD（屏幕二维信息显示设置选项）
框架下HUD是一种二维的屏幕显示信息，如玩家的生命值、分数信息等会在该类中进行绘制显示。
UE4中HUD、UMG、Slate之间的区别




参考：

[UE4 GamePlay架构（入门）_蜡笔小黑-CSDN博客](https://blog.csdn.net/weiweidunan/article/details/116190633)

[UE4的GamePlay框架概述_wjysg8408982的博客-CSDN博客](https://blog.csdn.net/wjysg8408982/article/details/112411867)



