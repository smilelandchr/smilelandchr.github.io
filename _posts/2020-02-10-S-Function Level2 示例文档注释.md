---
layout:     post
title:         2020-02-10-S-Function Level2 示例文档注释
subtitle:   
date:        2020-02-10
author:     SMILELAND
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Sfunction
    - level2
---

function msfuntmpl(block)
%MSFUNTMPL A Template for a MATLAB S-Function
%   The MATLAB S-function is written as a MATLAB function with the
%   same name as the S-function. Replace 'msfuntmpl' with the name
%   of your S-function.  

%   Copyright 2003-2018 The MathWorks, Inc.
  
%
% The setup method is used to setup the basic attributes of the
% S-function such as ports, parameters, etc. Do not add any other
% calls to the main body of the function.  
％设置方法用于设置S功能的基本属性，例如端口，参数等。请勿在功能主体上添加其他任何调用。
%   

setup(block);%设置模块
  
%endfunction

% Function: setup ===================================================
% Abstract:
%   Set up the S-function block's basic characteristics such as:%设定基础参数
%   - Input ports%输入端口
%   - Output ports%输出端口
%   - Dialog parameters%对话参数
%   - Options%选项
% 
%   Required         : Yes
%   C MEX counterpart: mdlInitializeSizes
%
function setup(block)

  % Register the number of ports.设置输入输出端口数
  block.NumInputPorts  = 1;
  block.NumOutputPorts = 1;
  
  % Set up the port properties to be inherited or dynamic.将端口属性设置为继承或动态。以指示输入和输出端口从模型继承其已编译属性（尺寸，数据类型，复杂度和采样模式）
  block.SetPreCompInpPortInfoToDynamic;%设定输入端口属性为动态
  block.SetPreCompOutPortInfoToDynamic;%设定输出端口属性为动态

  % Override the input port properties.覆盖输入端口属性
  block.InputPort(1).DatatypeID  = 0;  % double
  block.InputPort(1).Complexity  = 'Real';
  
  % Override the output port properties.覆盖输出端口属性
  block.OutputPort(1).DatatypeID  = 0; % double
  block.OutputPort(1).Complexity  = 'Real';

  % Register the parameters.初始化对话框参数
  block.NumDialogPrms     = 3;%初始化三个S函数对话框参数
  block.DialogPrmsTunable = {'Tunable','Nontunable','SimOnlyTunable'};%参数是否可变
  
  % Set up the continuous states.设定具有连续状态变量个数
  block.NumContStates = 1;

  % Register the sample times.设定采样时间
  %  [0 offset]            : Continuous sample time连续的采样时间
  %  [positive_num offset] : Discrete sample time离散的采样时间
  %
  %  [-1, 0]               : Inherited sample time继承的采样时间
  %  [-2, 0]               : Variable sample time可变采样时间
  block.SampleTimes = [0 0];%设定上面的采样时间，设置为[-1 0]即制定S函数具有继承的采样时间
  
  % -----------------------------------------------------------------
  % Options选相
  % -----------------------------------------------------------------
  % Specify if Accelerator should use TLC or call back to the 
  % MATLAB file
  %指定Accelerator是否使用TLC还是调回MATLAB文件
  block.SetAccelRunOnTLC(false);%否，不使用
  
  % Specify the block's operating point compliance. The block operating 
  % point is used during the containing model's operating point save/restore)
  %指定块的工作点符合性。 在包含模型的工作点保存/恢复期间使用块工作点）
  % The allowed values are:允许的值有
  %   'Default' : Same the block's operating point as of a built-in block与内置块相同的块的工作点
  %   'UseEmpty': No data to save/restore in the block operating point没有数据要在块操作点中保存/恢复
  %   'Custom'  : Has custom methods for operating point save/restore具有用于保存/还原工作点的自定义方法
  %                 (see GetOperatingPoint/SetOperatingPoint below)
  %   'Disallow': Error out when saving or restoring the block operating point.保存或恢复块操作点时出错。
  block.OperatingPointCompliance = 'Default';%默认
  
  % -----------------------------------------------------------------
  % The MATLAB S-function uses an internal registry for all
  % block methods. You should register all relevant methods
  % (optional and required) as illustrated below. You may choose
  % any suitable name for the methods and implement these methods
  % as local functions within the same file.
  %MATLAB S函数将内部注册表用于所有块方法。 您应该注册所有相关方法（可选和必需），如下所示。 您可以为这些方法选择任何合适的名称，并将这些方法实现为同一文件中的本地函数。
  % -----------------------------------------------------------------
   
  % -----------------------------------------------------------------
  % Register the methods called during update diagram/compilation.注册在更新图/编译期间调用的方法。
  % -----------------------------------------------------------------
  
  % 
  % CheckParameters:检查参数
  %   Functionality    : Called in order to allow validation of the
  %                      block dialog parameters. You are 
  %                      responsible for calling this method
  %                      explicitly at the start of the setup method.
  % 功能：调用以允许验证块对话框参数。 您有责任在setup方法开始时显式调用此方法。
  %   C MEX counterpart: mdlCheckParameters
  %
  block.RegBlockMethod('CheckParameters', @CheckPrms);

  %
  % SetInputPortSamplingMode:
  %   Functionality    : Check and set input and output port 
  %                      attributes and specify whether the port is operating 
  %                      in sample-based or frame-based mode
  %   C MEX counterpart: mdlSetInputPortFrameData.
  %   (The DSP System Toolbox is required to set a port as frame-based)
  %
  block.RegBlockMethod('SetInputPortSamplingMode', @SetInpPortFrameData);
  
  %
  % SetInputPortDimensions:
  %   Functionality    : Check and set the input and optionally the output
  %                      port dimensions.
  %   C MEX counterpart: mdlSetInputPortDimensionInfo
  %
  block.RegBlockMethod('SetInputPortDimensions', @SetInpPortDims);

  %
  % SetOutputPortDimensions:
  %   Functionality    : Check and set the output and optionally the input
  %                      port dimensions.
  %   C MEX counterpart: mdlSetOutputPortDimensionInfo
  %
  block.RegBlockMethod('SetOutputPortDimensions', @SetOutPortDims);
  
  %
  % SetInputPortDatatype:
  %   Functionality    : Check and set the input and optionally the output
  %                      port datatypes.
  %   C MEX counterpart: mdlSetInputPortDataType
  %
  block.RegBlockMethod('SetInputPortDataType', @SetInpPortDataType);
  
  %
  % SetOutputPortDatatype:
  %   Functionality    : Check and set the output and optionally the input
  %                      port datatypes.
  %   C MEX counterpart: mdlSetOutputPortDataType
  %
  block.RegBlockMethod('SetOutputPortDataType', @SetOutPortDataType);
  
  %
  % SetInputPortComplexSignal:
  %   Functionality    : Check and set the input and optionally the output
  %                      port complexity attributes.
  %   C MEX counterpart: mdlSetInputPortComplexSignal
  %
  block.RegBlockMethod('SetInputPortComplexSignal', @SetInpPortComplexSig);
  
  %
  % SetOutputPortComplexSignal:
  %   Functionality    : Check and set the output and optionally the input
  %                      port complexity attributes.
  %   C MEX counterpart: mdlSetOutputPortComplexSignal
  %
  block.RegBlockMethod('SetOutputPortComplexSignal', @SetOutPortComplexSig);
  
  %
  % PostPropagationSetup:
  %   Functionality    : Set up the work areas and the state variables. You can
  %                      also register run-time methods here.
  %   C MEX counterpart: mdlSetWorkWidths
  %
  block.RegBlockMethod('PostPropagationSetup', @DoPostPropSetup);

  % -----------------------------------------------------------------
  % Register methods called at run-time
  % -----------------------------------------------------------------
  
  % 
  % ProcessParameters:
  %   Functionality    : Call to allow an update of run-time parameters.
  %   C MEX counterpart: mdlProcessParameters
  %  
  block.RegBlockMethod('ProcessParameters', @ProcessPrms);

  % 
  % InitializeConditions:
  %   Functionality    : Call to initialize the state and the work
  %                      area values.
  %   C MEX counterpart: mdlInitializeConditions
  % 
  block.RegBlockMethod('InitializeConditions', @InitializeConditions);
  
  % 
  % Start:
  %   Functionality    : Call to initialize the state and the work
  %                      area values.
  %   C MEX counterpart: mdlStart
  %
  block.RegBlockMethod('Start', @Start);

  % 
  % Outputs:
  %   Functionality    : Call to generate the block outputs during a
  %                      simulation step.
  %   C MEX counterpart: mdlOutputs
  %
  block.RegBlockMethod('Outputs', @Outputs);

  % 
  % Update:
  %   Functionality    : Call to update the discrete states
  %                      during a simulation step.
  %   C MEX counterpart: mdlUpdate
  %
  block.RegBlockMethod('Update', @Update);

  % 
  % Derivatives:
  %   Functionality    : Call to update the derivatives of the
  %                      continuous states during a simulation step.
  %   C MEX counterpart: mdlDerivatives
  %
  block.RegBlockMethod('Derivatives', @Derivatives);
  
  % 
  % Projection:
  %   Functionality    : Call to update the projections during a
  %                      simulation step.
  %   C MEX counterpart: mdlProjections
  %
  block.RegBlockMethod('Projection', @Projection);
  
  % 
  % SimStatusChange:
  %   Functionality    : Call when simulation enters pause mode
  %                      or leaves pause mode.
  %   C MEX counterpart: mdlSimStatusChange
  %
  block.RegBlockMethod('SimStatusChange', @SimStatusChange);
  
  % 
  % Terminate:
  %   Functionality    : Call at the end of a simulation for cleanup.
  %   C MEX counterpart: mdlTerminate
  %
  block.RegBlockMethod('Terminate', @Terminate);

  %
  % GetOperatingPoint:
  %   Functionality    : Return the operating point of the block.
  %   C MEX counterpart: mdlGetOperatingPoint
  %
  block.RegBlockMethod('GetOperatingPoint', @GetOperatingPoint);
  
  %
  % SetOperatingPoint:
  %   Functionality    : Set the operating point data of the block using
  %                       from the given value.
  %   C MEX counterpart: mdlSetOperatingPoint
  %
  block.RegBlockMethod('SetOperatingPoint', @SetOperatingPoint);

  % -----------------------------------------------------------------
  % Register the methods called during code generation.
  % -----------------------------------------------------------------
  
  %
  % WriteRTW:
  %   Functionality    : Write specific information to model.rtw file.
  %   C MEX counterpart: mdlRTW
  %
  block.RegBlockMethod('WriteRTW', @WriteRTW);
%endfunction

% -------------------------------------------------------------------
% The local functions below are provided to illustrate how you may implement
% the various block methods listed above.
%提供下面的局部功能是为了说明如何实现上面列出的各种块方法。
% -------------------------------------------------------------------

function CheckPrms(block)
  
  a = block.DialogPrm(1).Data;
  if ~isa(a, 'double')
    me = MSLException(block.BlockHandle, message('Simulink:blocks:invalidParameter'));
    throw(me);
  end
  
%endfunction

function ProcessPrms(block)

  block.AutoUpdateRuntimePrms;
 
%endfunction

function SetInpPortFrameData(block, idx, fd)
  
  block.InputPort(idx).SamplingMode = fd;
  block.OutputPort(1).SamplingMode  = fd;
  
%endfunction

function SetInpPortDims(block, idx, di)
  
  block.InputPort(idx).Dimensions = di;
  block.OutputPort(1).Dimensions  = di;

%endfunction

function SetOutPortDims(block, idx, di)
  
  block.OutputPort(idx).Dimensions = di;
  block.InputPort(1).Dimensions    = di;

%endfunction

function SetInpPortDataType(block, idx, dt)
  
  block.InputPort(idx).DataTypeID = dt;
  block.OutputPort(1).DataTypeID  = dt;

%endfunction
  
function SetOutPortDataType(block, idx, dt)

  block.OutputPort(idx).DataTypeID  = dt;
  block.InputPort(1).DataTypeID     = dt;

%endfunction  

function SetInpPortComplexSig(block, idx, c)
  
  block.InputPort(idx).Complexity = c;
  block.OutputPort(1).Complexity  = c;

%endfunction 
  
function SetOutPortComplexSig(block, idx, c)

  block.OutputPort(idx).Complexity = c;
  block.InputPort(1).Complexity    = c;

%endfunction 
    
function DoPostPropSetup(block)
  block.NumDworks = 2;
  
  block.Dwork(1).Name            = 'x1';
  block.Dwork(1).Dimensions      = 1;
  block.Dwork(1).DatatypeID      = 0;      % double
  block.Dwork(1).Complexity      = 'Real'; % real
  block.Dwork(1).UsedAsDiscState = true;
  
  block.Dwork(2).Name            = 'numPause';
  block.Dwork(2).Dimensions      = 1;
  block.Dwork(2).DatatypeID      = 7;      % uint32
  block.Dwork(2).Complexity      = 'Real'; % real
  block.Dwork(2).UsedAsDiscState = true;
  
  % Register all tunable parameters as runtime parameters.
  block.AutoRegRuntimePrms;

%endfunction

function InitializeConditions(block)

block.ContStates.Data = 1;

%endfunction

function Start(block)

  block.Dwork(1).Data = 0;
  block.Dwork(2).Data = uint32(1); 
   
%endfunction

function WriteRTW(block)
  
   block.WriteRTWParam('matrix', 'M',    [1 2; 3 4]);
   block.WriteRTWParam('string', 'Mode', 'Auto');
   
%endfunction

function Outputs(block)
  
  block.OutputPort(1).Data = block.Dwork(1).Data + block.InputPort(1).Data;
  
%endfunction

function Update(block)
  
  block.Dwork(1).Data = block.InputPort(1).Data;
  
%endfunction

function Derivatives(block)

block.Derivatives.Data = 2*block.ContStates.Data;

%endfunction

function Projection(block)

states = block.ContStates.Data;
block.ContStates.Data = states+eps; 

%endfunction

function SimStatusChange(block, s)
  
  block.Dwork(2).Data = block.Dwork(2).Data+1;    

  if s == 0
    disp('Pause in simulation.');
  elseif s == 1
    disp('Resume simulation.');
  end
  
%endfunction
    
function Terminate(block)

disp(['Terminating the block with handle ' num2str(block.BlockHandle) '.']);

%endfunction
 
function operPointData = GetOperatingPoint(block)
% package the Dwork data as the entire operating point of this block
operPointData = block.Dwork(1).Data;

%endfunction

function SetOperatingPoint(block, operPointData)
% the operating point of this block is the Dwork data (this method 
% typically performs the inverse actions of the method GetOperatingPoint)
block.Dwork(1).Data = operPointData;

%endfunction
