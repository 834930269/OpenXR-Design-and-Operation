# 2 概述

OpenXR是一个分层的体系架构,由以下几层组成.  
- OpenXR Application (应用层)
- OpenXR Loader (中间层)
- OpenXR API Layers (插件层)
- OpenXR Runtimes (硬件层)

本文适用于Windows与Linux操作系统.

首先,我们先来了解一下OpenXR环境的整体结构.一个OpenXR应用程序是在一个执行链上运行的,接口直接与`OpenXR loader(后面记为loader)`交互.`loader`用于转换,检测,加载和交互和任意数量的OpenXR runtimes和OpenXR API Layers.每个OpenXR runtime控制一个完整的VR/XR/MR系统(比如Pico/hololens/oculus/华为 Glass).这个loader也可以在Runtime与application注入一定数量的API layers来增加系统可用的行为(额外的层,可以理解为插件).因此,任何OpenXR命令都可能涉及在许多不同模块中执行代码，包括加载程序、API层和运行时。

![](https://kheresy.files.wordpress.com/2020/07/openxr-loader.png?w=809)

## 2.1 谁应该阅读该文档

这个文档主要是为了开发OpenXR runtimes或者API layers,也希望可以为OpenXR loader做出一定贡献;当然,也可以帮助每个想要理解OpenXR的人.

[OpenXR specification](https://www.khronos.org/registry/OpenXR/ "OpenXR specification")应作为理解OpenXR API的主要手段，并且应注意不要与该文档的任何元素冲突。在本文档和OpenXR API规范不同的任何情况下，必须将[OpenXR specification](https://www.khronos.org/registry/OpenXR/ "OpenXR specification")定义的行为视为正确的标准。

## 2.2 OpenXR Loader

Loader对于检测、公开和可能加载系统上任何可用的OpenXR运行时(OpenXR runtimes)或API层至关重要。设置完成后，加载程序还负责管理OpenXR命令到每个组件的正确调度。

本文件旨在概述加载器和以下设备之间的必要接口：

- OpenXR Applications
- OpenXR API Layers
- OpenXR Runtimes

此外,这个文档也覆盖了大量OpenXR loader的内部设计.

### 2.2.1 OpenXR Loader的目的

loader被设计实现以下几个目标:

- 必须支持用户计算平台上的一个或者多个基于OpenXR的OpenXR runtime
- 必须支持OpenXR API layers(可配置模块需要由应用,开发者,或者标准系统设置开启)
- 必须努力减少对OpenXR应用程序的总体内存和性能的影响。

## 2.3 OpenXR API Layers

API层是增强OpenXR系统的可选组件。他们可以在从应用程序到运行时的过程中截取、计算、修改和插入现有的OpenXR命令。API层被实现为以各种方式（包括通过应用程序请求）启用的库。在`xrCreateInstance`调用期间，OpenXR系统将会启用所有API层。每个API层可以选择钩住（截获）任何OpenXR命令，而OpenXR方法又可以被忽略或增强。  
API层不需要截取所有OpenXR命令，只需要截取它想要的那些命令。  

关于API layers可能会用到的一些特性的示例:

- 验证API的使用情况
- 添加和执行API的调试和追踪功能
- 截取并过滤应OpenXR Application和OpenXR runtime之间的信息

由于API层是可配置的,你可以选择开启API层对您的应用进行Debug,但是当您的应用关闭时,您必须释放您所有的API layer.

更多关于API layer的信息将在后面的章节介绍

## 2.4 OpenXR Runtimes

OpenXR在多数的运行时上工作,支持多个或一个设备.每个OpenXR runtimes控制一个完整的VR/MR/AR系统.一个系统上可能安装多个runtimes,对于任何时间内正在执行的Runtime,OpenXR loader会找到当前系统中正在执行的runtime并且加载它.对应设备上的runtime会找到对应的在 `XrCreateInstance` 期间执行的 `XrInstance`.(比如Oculus找到Oculus的instance)

## 2.5. OpenXR Objects

用于定义模型的控制与行为.

## 2.6 OpenXR Call Chains(调用链)

当一个可配置的API layer已经被加载,这个loader将连接到另一个包含每一个被这个API layer选择的 `call chain`(调用链).在`xrCreateInstance`时对于每个OpenXR这个loader的指令初始化所有被开启的API layer和创建的调用链.	  
结果分发表的每个条目都指向该链的第一个元素。加载器为所创建的`XrInstance`构建一个单独的分发表。  