---
layout:     post
title:         2020-02-10-Simulink基于level2 的s-function函数编写
subtitle:   Timestwo的一个简单的说明
date:        2020-02-10
author:     SMILELAND
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Sfunction
    - Level2
    - 应用说明
---

## Simulink基于level2 的s-function函数编写

在用模块搭建时会遇到意想不到的问题，比如解微分方程模块，一旦出现分母为0情况就会报错，而自己编写S-Function就可以提前将为0的分母替换为极小值，既不会报错又基本不会影响计算结果。level2的函数可以配置多输入多输出。同时，MEX的S-Function可以实现更多的回调函数。

### 1. S-Function Bulder

S-Function Bulder可以简化S函数构建流程，但是不能生成多余1个的输入与输出，对于我来说不适用。

### 2. 一个基本的C MEX S-Function范例

一个timestwo范例，2倍。

![enter description here](https://i.loli.net/2020/02/10/kyugBJ98zXRT7S6.png)

timestwo S-Function包含的回调函数如下图。

![enter description here](https://i.loli.net/2020/02/10/LnVGfCbIAiXS2NP.png)

timestwo.c 的内容如下：

``` javascript
#define S_FUNCTION_NAME timestwo
#define S_FUNCTION_LEVEL 2
#include “simstruc.h”
static void mdlInitializeSizes ( SimStruct *S )
{
	ssSetNumSFcnParams( S， 0 )；
	if (ssGetNumSFcnParams(S) != ssGetSFcnParamsCount (S)) {
		return； /* Parameter mismatch will be reported by Simulink */
	}
	if ( !ssSetNumInputPorts( S，1 )) return；
	ssSetInputPortWidth (S，0，DYNAMICALLY_SIZED )；
	ssSetInputPortDirectFeedThrough (S，0，1)；
	if ( !ssSetNumOutputPorts (S，1)) return；
	ssSetOutputPortWidth ( S，0，DYNAMICALLY_SIZED )；
	ssSetNumSampleTimes ( S，1)；
	/* Take care when specifying exception free code - see sfuntmpl.doc */
	ssSetOptions ( S，SS_OPTION_EXCEPTION_FREE_CODE )；
}
	static void mdlInitializeSampleTimes ( SimStruct *S )
{
	ssSetSampleTime (S，0，INHERITED_SAMPLE_TIME )；
	ssSetOffsetTime ( S，0，0.0 )；
}
	static void mdlOutputs(SimStruct *S, int_T tid)
{
	int_T I；
	InputRealPtrsType uPtrs = ssGetInputPortRealSignalPtrs ( S，0 )；
	real_T *y = ssGetOutputPortRealSignal (S，0 )；
	int_T width = ssGetOutputPortWidth ( S，0 )；
	for ( i=0；i<width；i++ ) {
		*y++ = 2.0 *( *uPtrs[ I ] )；
	}
}
static void mdlTerminate (SimStruct *S) { }

#ifdef MATLAB_MEX_FILE /* Is this file being compiled as a MEX-file? */
#include “simulink.c” /* MEX-file interface mechanism */
#else
#include “cg_sfun.h” /* Code generation registration function */
#endif
```

该范例包括三个部分：

 - 定义与包含
 - 回调函数的实现
 - Simulink接口

#### 2.1 定义与包含

该范例以一下的定义开头：

``` javascript
#define S_FUNCTION_NAME timestwo
#define S_FUNCTION_LEVEL 2
#include "simstruc.h"
```

第一条指定S-function的名字(timestwo)，第二条指定了该S-function是按照level2的格式进行编写的。然后包含的以头文件“simstruc.h”。

#### 2.2 回调函数的实现

##### mdlInitializeSizes

Simulink调用**mdlInitializeSizes**来获取输入端口和输出端口的数量、端口宽度等信息。

- 无参数——S-function对话框的S-function parameters区必须为空。如果在此处输入了任何参数，Simulink将报告参数不匹配信息。
- 一个输入端口与一个输出端口——输入和输出端口的宽度是动态的。
- 一个采样时间——timestwo范例在程序**mdlInitializeSampleTimes**中指定了采样时间的实际值。
- 代码无异常检测——制定**exception-free**代码可以提高S-function的执行速度。在指定该选项时必须特别小心。

##### mdlInitializeSampleTimes

调用**mdlInitialzeSampleTimes**来设置S-function的采样时间。

##### mdlOutputs

在每个采样时间步长内，Simulink调用mdlOutpus来计算块的输出。

timestwo的mdlOutputs函数使用了SimStruct的一个宏：

	InputRealPtrsType uPtrs = ssGetInputPortRealSignalPtrs（S,0）;

来获取输入信号。该宏返回一个向量的指针，必须使用 *uPtrs[i]来访问它。

timestwo的mdlOutputs函数使用了SimStruct的一个宏：

	real_T   *y = ssGetOutputPortRealSignal(S,0);
	
来访问输出信号。该宏返回一个包含了输出向量的指针。

S-function使用int_T   width = ssGetOutputPortWidth(S,0)来获取块传递的信号宽度。最后，S-function采用循环通过输入来计算输出。

##### mdlTerminate

执行仿真结束时的任务。这是一个托管S-function程序。但是，timestwo S-function不需要执行任何终止动作，所以该程序是空的。

##### Building Timestwo范例

要将此S-function结合到Simulink中，在MATLAB命令行输入下面的命令：

	mex timestwo.c
	
mex命令将timestwo.c编译和链接以生成一个可动态下载的可执行文件，供Simulink使用。

最后的可执行文件是一个MEX S-funciton文件，其中，MEX
代表了MATLAB EXecutable（可执行文件）。MEX文件的扩展名根据不同的平台而有所不同，例如，在Microsoft Windows中，MEX文件的扩展名为.dll。

