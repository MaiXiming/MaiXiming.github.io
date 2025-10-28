---
layout: single
title: "具身智能概念"
date: 2025-10-17 10:00:00 +0800
categories: [embodied]
tags: [embodied ai]
toc: true
---

### VLM VLN VLA
- Vision-Language Model (VLM): 看图+理解语言 -> 输出语言或多模态表征（如描述、回答、匹配）。
- Vision-and-Language Navigation (VLN): 看环境中视觉信息+理解语言目标指令 -> 输出长程导航动作（向前、向左、停止等low level动作，应该不指输出轨迹）以完成目标。用于无人机/小车等导航任务
- Vision-Language-Action (VLA): 看环境中视觉信息+理解语言目标指令 -> 输出长程具体动作以完成目标。用于机器人完成抓取等任务。

三者关系：VLM是VLN/VLA的基础，VLN属于VLA。因为vln输出的动作空间action space仅限于走路，比如无人车导航的前进/左右转/停止等。一旦涉及机器人的动作，就是vla。导航可以看作是动作的一种。但机器人操控认为是vla。比如输出末端位姿变化，虽然不是关节角度，但是依然输出动作，所以是vla而不是vln。

### GraspNet / AnyGrasp
