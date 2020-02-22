---
layout:     post
title:         2020-02-22-如何在matlab sfunction 函数中调用自己写的函数？
subtitle:   
date:        2020-02-22
author:     SMILELAND
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - sfunction
    - 自定义函数
---

引用

![enter description here](https://i.loli.net/2020/02/22/9uEwW4YImxTGhSH.png)

自己编写了一个s函数，有几个参数引用了自己写的几个函数，在脚本中可以正确运行，但在写成s函数，进行 simulink 仿真的时候，已知提示“too many input auguments”,不知道怎么回事。

经过调试发现，把那几个参数换成常值，simulink 可以正常运行。说明：s函数中，不能直接调用自己写的函数。

那么问题来了：

在用 sfuntmpl 模板用m语言写的s函数中，不能直接调用自己写的函数，怎么办？

具体的解决办法是：

> 1、打开主页上的‘set path’选项，把自己写的函数所在的文件夹加到搜素路径中。

> 2、在matlab的安装文件路径 X:\Program Files\MATLAB\R2012\toolbox\ 下，新建一个文件夹 ‘myTool’。把自己写的函数复制到此文件夹下，然后把次文件夹加到路径中。

任选一种方法即可。个人比较推荐2。