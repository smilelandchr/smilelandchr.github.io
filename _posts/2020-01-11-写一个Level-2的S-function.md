---
layout:     post
title:         2020-01-11-写一个Level-2的S-function
subtitle:   Matlab官方教程
date:        2020-01-11
author:     SMILELAND
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - sfunction
    - level2
    - 教程
    - matlab
    - simulink
---

## Write Level-2 MATLAB S-Functions

From：https://ww2.mathworks.cn/help/simulink/sfg/writing-level-2-matlab-s-functions.html

### About Level-2 MATLAB S-Functions

使用Level-2MATLAB®S函数API，您可以使用MATLAB语言来创建具有多个输入和输出端口的自定义块，并能够处理Simulink®模型产生的任何类型的信号，包括任何数据的矩阵和帧信号 类型。  2级MATLAB S函数API与创建C MEX S函数的API紧密对应。 创建C MEX S函数的许多文档也适用于Level-2 MATLAB S函数。 为避免重复，本节着重于提供特定于编写Level-2 MATLAB S函数的信息。

Level-2 MATLAB S-function是MATLAB函数，用于定义在Simulink模型中引用MATLAB函数的Level-2 MATLAB S-Function块实例的属性和行为。  MATLAB函数本身包含一组回调方法（请参见Level 2 MATLAB S-Function回调方法），Simulink引擎在更新或仿真模型时会调用这些方法。 回调方法执行初始化和计算由S函数定义的块的输出的实际工作。
	
为了完成这些任务，引擎将运行时对象作为参数传递给回调方法。 运行时对象有效地充当了S功能块的MATLAB代理，从而允许回调方法在仿真或模型更新期间设置和访问块属性。

### About Run-Time Objects（关于运行对象）

Simulink引擎调用Level-2 MATLAB S函数回调方法时，会将*Simulink.MSFcnRunTimeBlock*类的实例作为参数传递给该方法。 该实例被称为S-Function块的运行时对象，对于Level-2 MATLAB S-function回调方法具有相同的目的，而SimStruct结构则用于C MEX S-function回调方法。 该对象使该方法能够提供并获得有关块端口，参数，状态和工作矢量的各种元素的信息。 该方法通过获取或设置块运行时对象的属性或调用方法来执行此操作。 有关获取和设置运行时对象属性以及调用运行时对象方法的信息，请参见Simulink.MSFcnRunTimeBlock类的文档。

运行时对象不支持MATLAB稀疏矩阵。 例如，如果变量块是运行时对象，则Level-2 MATLAB S函数中的以下行会产生错误：

    block.Outport(1).Data = speye(10);
	
speye命令形成一个稀疏身份矩阵。

### Level-2 MATLAB S-Function Template（Level 2 MATLAB S函数模板）

使用基本的Level-2 MATLAB S函数模板*msfuntmpl_basic.m*来开始创建新的Level-2 MATLAB S函数。 该模板包含由Level-2 MATLAB S-function API定义的所需回调方法的框架实现。 要编写更复杂的S函数，请使用带注释的模板*msfuntmpl.m*。

要创建MATLAB S函数，请制作一个模板副本，并根据需要编辑该副本，以反映您正在创建的S函数的所需行为。 以下两节描述了MATLAB代码模板的内容。 编写*Level-2 MATLAB S-Function*的示例部分介绍了如何编写对单位延迟建模的2级MATLAB S函数。

### Level-2 MATLAB S-Function Callback Methods（2级MATLAB S函数回调方法）

2级MATLAB S函数API定义了构成2级MATLAB S函数的回调方法的签名和一般用途。  S函数本身提供了这些回调方法的实现。 所述实现又确定块属性（例如，端口，参数和状态）和行为（例如，块输出作为时间的函数以及块输入，状态和参数）。 通过使用一组适当的回调方法创建S函数，您可以定义满足应用程序特定要求的块类型。

一个2级MATLAB S函数必须包括以下回调方法：

- 用于初始化基本S函数特征的设置函数
- 一个用于计算S函数输出的Outputs函数

根据S函数定义的块的要求，您的S函数可以包含其他方法。  Level 2 MATLAB S函数API定义的方法通常对应于C MEX S函数API定义的类似命名的方法。 有关在仿真过程中何时调用这些方法的信息，请参见Simulink Engine与CS功能交互中的“过程视图”。

下表列出了所有2级MATLAB S函数回调方法。

| Level-2 MATLAB Method         |
| ----------------------------- |
| setup                         |
| CheckParameters               |
| Derivatives                   |
| Disable                       |
| Enable                        |
| InitializeConditions          |
| Outputs                       |
| PostPropagationSetup          |
| ProcessParameters             |
| Projection                    |
| SetInputPortComplexSignal     |
| SetInputPortDataType          |
| SetInputPortDimensions        |
| SetInputPortDimensionsModeFcn |
| SetInputPortSampleTime        |
| SetOutputPortComplexSignal    |
| SetOutputPortDataType         |
| SetOutputPortDimensions       |
| SetOutputPortSampleTime       |
| SimStatusChange               |
| Start                         |
| Terminate                     |
| Update                        |

### Using the setup Method（使用设置方法）

Level 2 MATLAB S-function中设置方法的主体将初始化相应Level-2 MATLAB S-Function块的实例。 在这方面，设置方法类似于C MEX S函数实现的mdlInitializeSizes和mdlInitializeSampleTimes回调方法。 设置方法执行以下任务：

- 初始化块的输入和输出端口数。
- 设置这些端口的属性，例如维度，数据类型，复杂性和采样时间。
- 指定块采样时间。 有关如何指定有效采样时间的更多信息，请参见《使用Simulink》中的“指定采样时间”。
- 设置S功能对话框参数的数量。
- 通过将MATLAB S函数中的局部函数的句柄传递给S函数块的运行时对象的RegBlockMethod方法来注册S函数回调方法。 有关使用RegBlockMethod方法的信息，请参见Simulink.MSFcnRunTimeBlock的文档。

### Example of Writing a Level-2 MATLAB S-Function（一个例子）

以下步骤说明了如何编写简单的Level-2 MATLAB S函数。 如果适用，这些步骤包括模型msfcndemo_sfundsc2中使用的S函数示例msfcn_unit_delay.m中的示例。 所有代码行都将变量名称块用于S函数运行时对象。

1  将Level 2 MATLAB S函数模板msfuntmpl_basic.m复制到您的工作文件夹中。 如果在复制文件时更改了文件名，请将功能行中的功能名更改为相同的名称。

2  修改设置方法以初始化S函数的属性。 对于此示例：

- 将运行时对象的NumInputPorts和NumOutputPorts属性设置为1，以初始化一个输入端口和一个输出端口。
- 调用运行时对象的*SetPreCompInpPortInfoToDynamic*和*SetPreCompOutPortInfoToDynamic*方法，以指示输入和输出端口从模型继承其已编译属性（尺寸，数据类型，复杂度和采样模式）。
- 将运行时对象的InputPort的DirectFeedthrough属性设置为false，以指示输入端口没有直接馈通。 保留在模板文件副本中设置的所有其他输入和输出端口属性的默认值。 为Dimensions，DatatypeID和Complexity属性设置的值将覆盖使用SetPreCompInpPortInfoToDynamic和SetPreCompOutPortInfoToDynamic方法继承的值。
- 将运行时对象的NumDialogPrms属性设置为1，以初始化一个S函数对话框参数。
- 通过将运行时对象的SampleTimes属性的值设置为[-1 0]，指定S函数具有继承的采样时间。
- 调用运行时对象的RegBlockMethod方法来注册此S函数中使用的以下四个回调方法。

  - PostPropagationSetup
  - InitializeConditions
  - Outputs
  - Update

   从模板文件的副本中删除所有其他注册的回调方法。 在对RegBlockMethod的调用中，第一个输入参数是S函数API方法的名称，第二个输入参数是MATLAB S函数中关联的局部函数的函数句柄。

   下面这个msfcn_unit_delay.m的例子，描述上面所述的步骤：

``` vhdl
function setup(block)

%% Register a single dialog parameter
block.NumDialogPrms  = 1;

%% Register number of input and output ports
block.NumInputPorts  = 1;
block.NumOutputPorts = 1;

%% Setup functional port properties to dynamically
%% inherited.
block.SetPreCompInpPortInfoToDynamic;
block.SetPreCompOutPortInfoToDynamic;

%% Hard-code certain port properties
block.InputPort(1).Dimensions        = 1;
block.InputPort(1).DirectFeedthrough = false;

block.OutputPort(1).Dimensions       = 1;

%% Set block sample time to [0.1 0]
block.SampleTimes = [0.1 0];

%% Register methods
block.RegBlockMethod('PostPropagationSetup',@DoPostPropSetup);
block.RegBlockMethod('InitializeConditions',@InitConditions);
block.RegBlockMethod('Outputs',             @Output);  
block.RegBlockMethod('Update',              @Update);  
```

如果您的S函数需要连续状态，请使用运行时对象的NumContStates属性在设置方法中初始化连续状态的数量。 不要在设置方法中初始化离散状态。

3  在PostPropagationSetup方法中初始化离散状态。  2级MATLAB S函数将离散状态信息存储在DWork向量中。 在此示例中，模板文件中的默认PostPropagationSetup方法就足够了。

   msfcn_unit_delay.m中的以下PostPropagationSetup方法名为DoPostPropSetup，它初始化了一个名为x0的DWork向量
   
   function DoPostPropSetup(block)

``` mipsasm
%% Setup Dwork
  block.NumDworks = 1;
  block.Dwork(1).Name = 'x0'; 
  block.Dwork(1).Dimensions      = 1;
  block.Dwork(1).DatatypeID      = 0;
  block.Dwork(1).Complexity      = 'Real';
  block.Dwork(1).UsedAsDiscState = true;
```

如果您的S函数使用其他DWork向量，则也应在PostPropagationSetup方法中对其进行初始化（请参见在Level 2 MATLAB S函数中使用DWork向量）。

4  在InitializeConditions或Start回调方法中初始化离散和连续状态或其他DWork向量的值。 对于模拟开始时一次初始化的值，请使用Start回调方法。 每当重新启用包含S功能的已启用子系统时，请使用InitializeConditions方法获取需要重新初始化的值。

    对于此示例，使用InitializeConditions方法将离散状态的初始条件设置为S函数的dialog参数的值。 例如，msfcn_unit_delay.m中的InitializeConditions方法为：
	
``` oxygene
	function InitConditions(block)

  %% Initialize Dwork
  block.Dwork(1).Data = block.DialogPrm(1).Data;
```

对于具有连续状态的S函数，请使用ContStates运行时对象方法初始化连续状态数据。 例如：

``` lsl
block.ContStates.Data(1) = 1.0;
```

5  在Outputs回调方法中计算S函数的输出。 对于此示例，将输出设置为DWork向量中存储的离散状态的当前值。

    msfcn_unit_delay.m中的Outputs方法为：
	
``` oxygene
	function Output(block)

  block.OutputPort(1).Data = block.Dwork(1).Data;
```

6  对于具有连续状态的S函数，请使用“导数”回调方法计算状态导数。 运行时对象将派生数据存储在其“派生”属性中。 例如，以下行将第一状态导数设置为等于第一输入信号的值。

``` lsl
block.Derivatives.Data(1) = block.InputPort(1).Data;
```

本示例不使用连续状态，因此不实现“派生”回调方法。

7  在Update回调方法中更新所有离散状态。 对于此示例，将离散状态的值设置为第一输入信号的当前值。
    msfcn_unit_delay.m中的Update方法是：
	
``` oxygene
	function Update(block)

  block.Dwork(1).Data = block.InputPort(1).Data;
```

8  在Terminate方法中执行任何清除操作，例如清除变量或内存。 与C MEX S函数不同，Level-2 MATLAB S函数不需要具有Terminate方法。

有关其他回调方法的信息，请参见Level 2 MATLAB S函数回调方法。 有关运行时对象属性的列表，请参见Simulink.MSFcnRunTimeBlock和父类Simulink.RunTimeBlock的参考页。

### Instantiating a Level-2 MATLAB S-Function

要在模型中使用Level-2 MATLAB S-function，请将Level-2 MATLAB S-Functionblock的实例复制到模型中。 打开该块的“块参数”对话框，然后在S函数名称字段中输入实现S函数的MATLAB文件的名称。 如果您的S函数使用任何其他参数，请在“块参数”对话框的“参数”字段中以逗号分隔的列表形式输入参数值。

### Operations for Variable-Size Signals（可变大小信号的操作）

以下是对2级MATLAB S函数模板（msfuntmpl_basic.m）的修改以及允许您使用可变大小信号的其他操作。

``` vhdl
function setup(block)
% Register the properties of the output port
block.OutputPort(1).DimensionsMode = 'Variable';
block.RegBlockMethod('SetInputPortDimensionsMode',  @SetInputDimsMode);

function DoPostPropSetup(block)
%Register dependency rules to update current output size of output port a depending on
%input ports b and c
block.AddOutputDimsDependencyRules(a, [b c], @setOutputVarDims);

%Configure output port b to have the same dimensions as input port a
block.InputPortSameDimsAsOutputPort(a,b);

%Configure DWork a to have its size reset when input size changes.
block.DWorkRequireResetForSignalSize(a,true);

function SetInputDimsMode(block, port, dm)
% Set dimension mode
block.InputPort(port).DimensionsMode = dm;
block.OutputPort(port).DimensionsMode = dm;

function setOutputVarDims(block, opIdx, inputIdx)
% Set current (run-time) dimensions of the output
outDimsAfterReset = block.InputPort(inputIdx(1)).CurrentDimensions;
block.OutputPort(opIdx).CurrentDimensions = outDimsAfterReset;
```

### Generating Code from a Level-2 MATLAB S-Function（从2级MATLAB S函数生成代码）

为包含Level-2 MATLAB S函数的模型生成代码要求您提供相应的目标语言编译器（TLC）文件。 您不需要TLC文件即可加速包含Level-2 MATLAB S函数的模型。  Simulink Accelerator™软件以解释模式运行Level-2 MATLAB S函数。 但是，如果M文件S函数在模型参考中，则M文件S函数无法在加速模式下使用。 有关为MATLAB S函数编写TLC文件的更多信息，请参见内联S函数（Simulink Coder）和内联MATLAB File S函数（Simulink Coder）。

### MATLAB S-Function Examples

Level 2 MATLAB S函数示例提供了一组自文档模型，这些模型说明了Level 2 MATLAB S函数的用法。 在MATLAB命令提示符下输入sfundemos以查看示例。

### MATLAB S-Function Limitations

- 2级MATLAB S函数不支持过零检测。
- 您不能从2级MATLAB S函数中触发函数调用子系统。