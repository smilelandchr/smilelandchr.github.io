---
layout:     post
title:         2020-01-09-S-function level1 错误
subtitle:   during flag=3 call must be a real vector of length 1 
date:        2020-01-09
author:     SMILELAND
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - matlab
    - simulink
    - S-function
    - 错误
---

在编写S-function时报错。

> error occurred while running the simulation and the simulation was terminated
Caused by:
Output returned by S-function 'cassie' in 'basic9/Cassie arc model sfunction/S-Function' during flag=3 call must be a real vector of length 1 

可能的错误因素：

1. 输出元素个数和你的定义输出个数不同；
2. s-function的输入变量个数和与其连接的模块的输入变量个数不一致；
3. 还有一种可能是输出为无穷，也就是除以0了；
4. S-function的输入变量的维数不是1维的向量；


