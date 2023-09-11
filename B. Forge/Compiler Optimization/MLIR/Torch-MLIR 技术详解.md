# 概述

Torch-MLIR 作为一个编译器支持将pytorch生态转换到MLIR生态，本文基于Torch-MLIR [开源项目](https://github.com/llvm/torch-mlir "开源项目") 对它的主要工作流程，以及涉及到的[MLIR](https://mlir.llvm.org/ "MLIR")主要概念进行了分析说明
# 架构

![[Pasted image 20230803201746.png]]
上面的架构图说明从Pytorch生态有多条pipeline可以转换到MLIR的世界，本文主要介绍

TorchScript -> MLIR Converter -> Torch Dialect -> Linalg Tensor 这条pipeline

# IR转换详解

![[Pasted image 20230803202012.png]]
![[Pasted image 20230803202019.png]]
![[Pasted image 20230803202028.png]]
![[Pasted image 20230803202253.png]]
![[Pasted image 20230803202407.png]]
上图说明了Pytorch模型代码 ->TorchScript IR -> TorchDialect IR 的转换对应关系

## OpConversion代码分析

![[Pasted image 20230803202531.png]]
![[Pasted image 20230803202633.png]]
![[Pasted image 20230803202823.png]]
![[Pasted image 20230803203128.png]]
![[Pasted image 20230803203237.png]]
![[Pasted image 20230803203446.png]]
![[Pasted image 20230803203453.png]]
![[Pasted image 20230803203504.png]]

上图以OpConversionPattern代码为例，说明了TorchDialect IR 中的torch.aten.mm Op 转换到LinalgDialect Op的过程

## 核心时序

![[Pasted image 20230803203648.png]]
 Torch-MLIR从Python应用、初始化、MLIR Converion Framework注册与回调、Dialect & Op Convert Pass 的核心时序图如上
 
# Linalg Generic Op说明

理解Linalg Dialect的Op才能对Torch Dialect Op做对应的转换，而Linalg Generic Op又是最重要的一个Op，用一个详细的例子对它说明如下：

![[Pasted image 20230803204539.png]]
![[Pasted image 20230803204544.png]]
![[Pasted image 20230803204556.png]]
![[Pasted image 20230803204607.png]]
![[Pasted image 20230803204625.png]]
![[Pasted image 20230803204637.png]]
![[Pasted image 20230803204649.png]]
![[Pasted image 20230803204655.png]]
# Code Gen VS LibCall
![[Pasted image 20230803204708.png]]
Code Gen：用一堆IR生成另外一堆表达同样计算语义的IR

LibCall: 用一个API调用时序一堆IR的计算，比如用cublasSgemm 计算矩阵乘法

[原文出处]([【架构分析】Torch-MLIR 技术详解_HaoBBNuanMM的博客-CSDN博客](https://blog.csdn.net/HaoBBNuanMM/article/details/124385542))
