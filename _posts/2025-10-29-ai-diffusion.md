---
layout: single
title: "Diffusion - 原理待补充"
date: 2025-10-29 10:00:00 +0800
categories: [ai]
tags: [diffusion, module]
toc: true
---

### 理解
GAN和VAE都是从正态分布的采样（纯噪声）作为输入，通过一个网络一次生成样本（图片等），单次过于暴力；

Diffusion也是一个网络，也是从正态分布的采样（纯噪声）作为输入，但是网络不断输入输出（类似RNN将当前输出作为下一时刻输入），配合时间步逐步去噪的方式，最终生成无噪声的样本。


### 用法




### 原理与公式 - 待理解和补充