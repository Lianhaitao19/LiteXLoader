## 👓 项目架构分析

> `LiteXLoader`是一个基岩版官方服务端`Bedrock Delicated Server`（以下简称**BDS**）插件框架，提供强大的跨语言脚本插件支持能力和稳定的开发API支持。

正如项目介绍所言，LXL一个拥有跨语言、跨平台能力的BDS服务端插件框架。

### 跨语言开发能力

LXL开发之初的想法，便是依靠跨语言引擎的支持，整合多种脚本语言，并给予统一的开发接口，解决相关开发生态破碎的问题。  
实际项目中，使用腾讯开源项目`ScriptX`的跨语言引擎，在对接多种后端的同时，ScriptX导出了统一的C++接口。  
因此，LXL在对接BDS时仅维护了1套底层接口，和脚本引擎的对接相当方便。  
目前，`ScriptX`仍在发展之中，新的语言支持还需时日，让我们拭目以待。

<br>

### 模块化，兼容性

LXL最重要的思想，就是将各重要功能模块化，方便于后续的维护和子项目的升级。基于模块化的设计思想，LXL将底层加载器接口、多种脚本引擎后端、插件加载模块互相分离，保证各个部分可以单独维护升级，各模块之间尽量降低耦合度，方便进行修改和功能新增。

`LiteXLoader`的架构，可以从此图一眼看出：

![LiteXLoader架构图](Structure.png)

- `LiteLoader`加载器和`ScriptX`两部分提供重要的基础接口
- Kernel内核抽象层负责所有对`LiteLoader`API的调用、Hook函数调用以及对其他底层库函数的调用，并将他们各自的类型抽象为标准的变量类型和STL容器，将各自的接口封装，避免对底层项目的强依赖扩散到上层。
- API接口层为ScriptX脚本引擎提供API接口，引擎将API注入到脚本系统中，在脚本中即是调用相关的API接口完成和BDS的交互
- ScriptX完成对下层脚本引擎的统一抽象，在上层提供一致的接口，为跨语言脚本开发提供基础

同时，由于使用C/C++语言开发，`LiteLoader`+`LiteXLoader`的加载器组合在Linux上同样可以使用 **Wine** 来支持运行，各功能仍能正常工作，且运行性能显著高于Linux原版BDS。使用Linux的开发者们，不用担心平台兼容性的问题。

<br>

### 开源项目目录结构介绍

项目使用cmake构建系统构建。  
上述的架构落实到实际项目中，项目目录结构如图所示：

```
├───docs				# 文档目录
├───LiteXLoader
│   ├───engine			# 脚本引擎库目录
│   ├───include			# 头文件包含目录
│   ├───lib				# 依赖库目录
│   ├───LiteXLoader.Js	# Js项目目录（无源码）
│   ├───LiteXLoader.Lua	# Lua项目目录（无源码）
│   ├───Release			# DLL生成目录
│   ├───src				# [核心]源码目录
│   │   ├───API			# API接口层
│   │   └───Kernel		# 核心抽象层
│   └───test			# 测试
├───RELEASE				# 发布目录（用于GitHub Action）
└───ScriptX				# ScriptX源码目录
```

<br>

### 相关底层原理

关于如何从BDS获取底层接口，BDS各大插件框架的方法都大同小异：使用Hook。这里会涉及到很多Windows操作系统底层相关的知识，这也是BDS的C++插件难以开发的原因之一。  

Hook技术，指的是在BDS执行某个函数的时候，通过一些机制修改BDS地址空间中的代码，让其转而先执行一些开发者自己的代码，再返回到BDS执行它原本要执行的东西。这样一来，开发者就可以在BDS中注入自己的逻辑。  

我们可以针对这个函数推测其被调用的原因，并在自己的代码中对这些事件做一些响应，比如说调用其他的BDS函数，或者记录到数据库，或者执行某条命令，甚至是拦截这个函数以达到阻止这个事件发生的目的......总而言之，就是通过Hook来实现一些原版服务器所无法实现的行为。
