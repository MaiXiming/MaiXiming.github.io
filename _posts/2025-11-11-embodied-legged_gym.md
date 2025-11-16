---
layout: single
title: "legged_gym详解（未完成）"
date: 2025-11-11 10:00:00 +0800
categories: [embodied]
tags: [isaacgym]
toc: true
---

### 背景
最近需要看开源仓库，首先发现是基于isaacgym的，花了两天看了底层API，但是并没有什么用；后来发现是基于legged_gym和rsl_rl的，花1+1h快速了解概念后就想上手项目代码，但是看了两天还是云里雾里。所以今天系统将legged_gym仓库代码看了看，一切就清晰了起来。

学习流程：legged_gym配置环境（isaacgym np.float->float）；然后看README；然后看envs下的类的定义，从Base到LeggedRobot；最后看train.py & play.py；结束。

### 原理
一句话理解：legged_gym是利用isaacgym API写的上层应用代码，但是因为定义的框架很好所以后来大家都基于legged_gym做机器人的二次开发。比如Humanoid-gym。
通过legged_gym，我们可以实现Isaacgym中的环境搭建，机器人导入，机器人策略训练，与仿真环境的交互。

### 训练过程机制`train.py`
1. 运行`legged_gym/scripts/train.py`，根据参数`--task`指定任务名，执行训练。
2. 根据参数`--task`指定的名字，在`legged_gym/envs/__init__.py`中定义了该任务名关联的三个组成部分，共同定义了任务应该如何运行。
    - **环境类Env Class**: 通过`step()`函数定义了仿真世界运行的步进发展方式。(`base_task.py -> legged_robot.py > custom_robot.py (anymal.py)`，类名类似于`LeggedRobot()`)
    - **环境配置类 Env Config**: 定义了环境类中待定的相关参数，比如并行环境数量num_envs、仿真时间dt等。（`base_config.py -> legged_robot_config.py -> anymal_config.py`, 类名类似于`LeggedRobotCfg()`）
    - **训练配置类 Train Config**: 定义了机器人控制策略训练过程的相关参数，如batch_size、iterations等。（`base_config.py -> legged_robot_config.py -> anymal_config.py`, 类名类似于`LeggedRobotCfgPPO()`）
3. train.py中，`env, env_cfg = task_registry.make_env(name=args.task, args=args)`通过任务名寻找对应的1环境类和2环境配置类，创建1环境类对象`env`和2环境配置类对象`env_cfg`。
4. train.py中，`ppo_runner, train_cfg = task_registry.make_alg_runner(env=env, name=args.task, args=args)`，通过任务名寻找对应的3训练配置类，创建3训练配置类对象`train_cfg`，并且创建强化学习算法PPO`rsl_rl.runners.OnPolicyRunner`类对象`ppo_runner`，根据`train_cfg`设置PPO算法的相关配置。
5. train.py中，`ppo_runner.learn(num_learning_iterations=train_cfg.runner.max_iterations, init_at_random_ep_len=True)`，`learn()`方法运行[for loop: 获取observations，根据算法输出动作actions，传入env进行step()函数步进环境运行，更新obs/rewards，更新算法]。最终保存训练完成的算法模型为.pt文件。

### 框架理解
```
│  README.md
│  setup.py
├─legged_gym
│  │  __init__.py: 定义legged_gym路径
│  ├─envs
│  │  │  __init__.py: 定义任务str与关联的环境类、环境配置实例()、训练配置实例()。 # 注意：1为类本身，23为类的对象实例
│  │  ├─base
│  │  │      base_config.py: 环境配置基类 BaseConfig
│  │  │      base_task.py: 环境类基类 BaseTask
│  │  │      legged_robot.py: 环境类LeggedRobot, 继承BaseTask
│  │  │      legged_robot_config.py: 环境配置类LeggedRobotCfg, 继承类BaseConfig; 训练配置类LeggedRobotCfgPPO，继承类BaseConfig。
│  │  └─cassie
│  │          cassie.py: 环境类Cassie, 继承LeggedRobot
│  │          cassie_config.py: 环境配置类CassieRoughCfg, 继承类BaseConfig; 训练配置类CassieRoughCfgPPO，继承类BaseConfig。
│  ├─scripts
│  │      play.py: 测试机器人
│  │      train.py: 训练机器人
│  │
│  ├─tests
│  │      test_env.py: 测试环境类是否能正常step()
│  └─utils
│          helpers.py: 辅助函数，如get_args(), class_to_dict()等。
│          logger.py: 数据记录类Logger，log / print / plot功能。
│          math.py: 数学相关，比如向量坐标转换quat_apply_yaw, wrap_to_pi, torch_rand_sqrt_float
│          task_registry.py: 任务定义类TaskRegistery()，包括关联任务名和3个类register(), 根据任务名返回环境类get_task_class()、返回配置类对象get_cfgs()，make_env(), make_alg_runner()
│          terrain.py: 地形类Terrain（待补充：暂时用不到，所以没细看了）
│          __init__.py: import上面的函数等。
├─licenses: assets的相关版权信息
└─resources: 定义机器人对象的.urdf/.stl等文件。
```

### 代码理解
#### `BaseConfig` (`base_config.py`)

| 函数名/代码块 | 函数实现功能与输入/输出 | 函数内容详解 |
| :--- | :--- | :--- | 
| `__init__(self)`              | 初始化实例 | 执行`self.init_member_classes(self)` |
| `init_member_classes(obj)`    | 静态方法，实现代码即配置的功能，代替.yaml的作用。 | `XXXCfg`下定义的都是一个个类，正常来说需要依次实例化后，才能cfg.var1.var2；通过`for key in dir(obj); getattr(obj, key); setattr(obj, key, i_var)`,将当前类下的定义的类全部实例化，直接通过env_cfg.var1.var2来获取其值。 <br> 所以环境类中，可以直接`env_cfg.env.num_envs`来获取值，不需要再将env实例化后才能获取num_envs |

#### `BaseTask` (`base_task.py`)

| 函数名/代码块 | 函数实现功能与输入/输出 | 函数内容详解 |
| :--- | :--- | :--- |
| `__init__(self, cfg, ...)` | 初始化实例，包括创建环境和buffer<br>输入：cfg - 环境配置类; sim_params等<br>输出：无 | 创建`self.gym`使用isaacgym api <br> `self.xxx = cfg.subclass.xxx`将环境配置类定义的参数转为实例的属性 <br> 初始化各种buffers `self.obs_buf, self.rew_buf, self.episode_length_buf` <br> 创建仿真环境`self.crate_sim()`，并创建环境对象`self.sim`； <br> 设置观测相机`self.viewer` |
| `self.create_sim()` | 创建环境 | 无，在继承类`LeggedRobot`中定义 |
| `get_observations(self)` | 返回`self.obs_buf`<br>输入：无<br>输出: self.obs_buf - tensor，由`__init__`创建，由继承类LeggedRobot `compute_observations()`更新 | / |
| `get_priviledged_observations(self)` | 返回`self.priviledged_obs_buf`<br>输入：无<br>输出: self.priviledged_obs_buf - tensor，由`__init__`创建，由继承类LeggedRobot `compute_observations()`更新 | / |
| `reset_idx(self, env_ids)` | 将env中的robot重置状态<br>输入/输出：无 | 无，在继承类中定义 |
| `reset(self)` | 重置所有robot<br>输入：无 <br>输出：obs, priviledged_obs | 调用`self.reset_idx()`，并且`self.step(actions=0)`<br>不知道为啥要多个step? |
| `step(self, actions)` | 步进物理世界的发展<br>输入：actions - tensor，模型输出的动作<br>输出：无 | 无，在继承类中定义 |
| `render(self, sync_frame_time=True)` | 渲染仿真世界的画面，键盘中断检测，和sim退出检查 | `self.fetch_results(self.sim, True)`来获取GPU计算结果，`self.gym.step_graphics(self.sim)`来渲染画面 <br>self.gym.sync_frame_time(self.sim)是干嘛的？ |


#### `LeggedRobotCfg` (`legged_robot_config.py`)

| 函数名/代码块 | 函数实现功能与输入/输出 | 函数内容详解 |
| :--- | :--- | :--- | 
| --- | --- | --- |
| --- | --- | --- |
| --- | --- | --- |

#### `LeggedRobotCfgPPO` (`legged_robot_config.py`)

| 函数名/代码块 | 函数实现功能与输入/输出 | 函数内容详解 |
| :--- | :--- | :--- | 
| --- | --- | --- |
| --- | --- | --- |
| --- | --- | --- |

#### `LeggedRobot` (`legged_robot.py`)

| 函数名/代码块 | 函数实现功能与输入/输出 | 函数内容详解 |
| :--- | :--- | :--- | 
| --- | --- | --- |
| --- | --- | --- |
| --- | --- | --- |

#### `Cassie`简单理解

| 函数名/代码块 | 函数实现功能与输入/输出 | 函数内容详解 |
| :--- | :--- | :--- | 
| --- | --- | --- |
| --- | --- | --- |
| --- | --- | --- |


### 测试过程机制`play.py`

### 添加自己的机器人环境



总体逻辑：每个机器人任务，都需要定义一个task name，一个isaacgym运行的环境类LeggedRobot，一个配置环境参数类LeggedRobotCfg，一个配置算法参数类LeggedRobotCfgPPO。
- 由task_registry.register()建立上述4者的关联，从而在train.py/play.py中利用task name属性，在TaskRegistry.make_env()中建立三个类的对象。
- 参数类继承了BaseConfig, 通过getattr & setattr的方式实例化参数类中的所有类，这样做的好处是代码即配置，代替了yaml。
- LeggedRobotCfg的所有参数会传入LeggedRobot实例的self.env，从而在LeggedRobot的运行isaacgym-仿真sim过程中利用参数定义相关tensor/执行相关动作行为。
- LeggedRobot的所有方法基本用的都是isaacgym API，所以它就是定义了仿真sim的进行动作和状态。其实例用于rsl_rl.learn()中调用step()，来计算更新仿真世界的各种状态。所以step()最关键，所有的强化学习相关的内容都在这里，包括状态更新，模型输入输出，由PID转换为扭矩控制电机，添加扰动，计算reward等。其他如compute_observations()等函数都是为step()服务的。

具体需要对整个库进行分析，有空补充。