---
layout: single
title: "legged_gym项目概括(有空补充详解)"
date: 2025-11-11 10:00:00 +0800
categories: [embodied]
tags: [isaacgym]
toc: true
---

### 背景
最近需要看开源仓库，首先发现是基于isaacgym的，花了两天看了底层API，但是并没有什么用；后来发现是基于legged_gym和rsl_rl的，花1+1h快速了解概念后就想上手项目代码，但是看了两天还是云里雾里。所以今天系统将legged_gym仓库代码看了看，一切就清晰了起来。

学习流程：legged_gym配置环境（isaacgym np.float->float）；然后看README；然后看envs下的类的定义，从Base到LeggedRobot；最后看train.py & play.py；结束。

### 框架理解
legged_gym是利用isaacgym API写的上层应用代码，但是因为定义的框架很好所以后来大家都基于legged_gym做机器人的二次开发。比如Humanoid-gym。

总体逻辑：每个机器人任务，都需要定义一个task name，一个isaacgym运行的环境类LeggedRobot，一个配置环境参数类LeggedRobotCfg，一个配置算法参数类LeggedRobotCfgPPO。
- 由task_registry.register()建立上述4者的关联，从而在train.py/play.py中利用task name属性，在TaskRegistry.make_env()中建立三个类的对象。
- 参数类继承了BaseConfig, 通过getattr & setattr的方式实例化参数类中的所有类，这样做的好处是代码即配置，代替了yaml。
- LeggedRobotCfg的所有参数会传入LeggedRobot实例的self.env，从而在LeggedRobot的运行isaacgym-仿真sim过程中利用参数定义相关tensor/执行相关动作行为。
- LeggedRobot的所有方法基本用的都是isaacgym API，所以它就是定义了仿真sim的进行动作和状态。其实例用于rsl_rl.learn()中调用step()，来计算更新仿真世界的各种状态。所以step()最关键，所有的强化学习相关的内容都在这里，包括状态更新，模型输入输出，由PID转换为扭矩控制电机，添加扰动，计算reward等。其他如compute_observations()等函数都是为step()服务的。

具体需要对整个库进行分析，有空补充。