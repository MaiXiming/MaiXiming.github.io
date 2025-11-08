---
layout: single
title: "URDF"
date: 2025-11-08 10:00:00 +0800
categories: [embodied]
tags: [embodied, urdf]
toc: true
---

URDF是一个定义link, joint的xml格式文件，一起构成了机器人本体。

link就是一个个连杆，joint则是连接parent link / child link之间的关节。joint在urdf中没有物理属性，就是一个运动关系，在实际中则是用电机来实现。

逻辑关系：
- 组成：world - joint - pelvis - joint - other - joint - other。一个link只能属于一个joint的child，所以无法构成闭环连接。
- 每个link有一个坐标系，joint定义了坐标系变换方式，仿真中驱动joint，会改变child link的坐标系位姿，也就是控制child link的运动了。所以joint & child link 在urdf visualizer中是关联在一起的。
- link表征=坐标系+物理属性定义：每个link其实就是一个坐标系xyz，然后inertial/visual/collision的 <xyz/rpy> 定义了质心/视觉渲染模型（我们看到的link是加载.stl文件）/碰撞模型相对于坐标系的pose位姿。
- joint=plink&clink的坐标系相对位置+运动关系：joint中则是定义了child link坐标系相对于parent link坐标系的<xyz/rpy>，然后再由child link的信息来定义具体的link表现形式（质心/视觉/碰撞），以此类推。joint中的axis<xyz>则是定义旋转轴（child link坐标系）。最终再由仿真中在驱动转动$\theta$角度-这时候child link坐标系也转动了。
