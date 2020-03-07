---
layout:     post
title:         2020-03-07-在Level2的S函数中应用Dwork
subtitle:   Using DWork Vectors in Level-2 MATLAB S-Functions
date:        2020-03-07
author:     SMILELAND
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - sfunction
    - level
    - Dwork
---

以下步骤显示了如何在Level-2 MATLAB Sfunctions中初始化和使用DWork向量。 这些步骤使用S函数msfcn_unit_delay.m。

1. 在PostPropagationSetup方法中，初始化DWork向量的数量以及每个向量的属性。 例如，以下PostPropagationSetup回调方法配置一个DWork向量，用于存储离散状态。

``` matlab
function PostPropagationSetup(block)
%% Setup Dwork
block.NumDworks = 1;
block.Dwork(1).Name = 'x0';
block.Dwork(1).Dimensions = 1;
block.Dwork(1).DatatypeID = 0;
block.Dwork(1).Complexity = 'Real';
block.Dwork(1).UsedAsDiscState = true;
```

Simulink.BlockCompDworkData和父类Simulink.BlockData的参考页列出了可以为Level-2 MATLAB Sfunction DWork向量设置的属性。

2. 在Start或InitializeConditions方法中初始化DWork向量值。 将Start方法用于仅在模拟开始时初始化的值。 每当重新启用包含S功能的已禁用子系统时，请使用InitializeConditions方法获取需要重新初始化的值。

例如，下面的InitializeConditions方法将在上一步中配置的DWork向量的值初始化为第一个S函数对话框参数的值。

``` matlab
function InitializeConditions(block)
%% Initialize Dwork
block.Dwork(1).Data = block.DialogPrm(1).Data;
```

3. 在“Outputs”，“Update”等方法中，根据需要使用或更新DWork矢量值。 例如，以下Outputs方法将S函数输出设置为等于DWork向量中存储的值。 然后，Update方法将DWork向量值更改为第一个S函数输入端口的当前值。

``` matlab
%% Outputs callback method
function Outputs(block)
block.OutputPort(1).Data = block.Dwork(1).Data;
%% Update callback method
function Update(block)
block.Dwork(1).Data = block.InputPort(1).Data;
```

> 2级MATLAB S函数不支持MATLAB稀疏矩阵。 因此，您不能将稀疏矩阵分配给DWork向量的值。 例如，以下代码行会产生错误
> `block.Dwork(1).Data = speye(10)`
> speye命令生成一个稀疏的恒等矩阵。

