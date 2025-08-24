---
title: Psychopy安装和使用
date: 2018-11-09 21:03:15
updated: 2022-06-04 14:49:00
permalink: /psychopy-install/
categories: [psychopy,]
tags: [psychopy,]
---

<!-- toc -->
Psychopy是基于Python的心理学实验设计软件，由英国诺丁汉大学的Jon Peirce主持开发。Psychopy结合了OpenGL的图形优势和Python的语法特性，给科学家们提供了快速构建高性能的图形刺激界面的工具。<!--more-->

## 我为什么选择Psychopy
在我研究生阶段，我做脑机接口实验编写刺激界面的工具主要是Matlab平台的Psychtoolbox。很早之前，我也用过e-Prime，但很快就放弃了。e-Prime提供GUI界面，简单易学，但是无法设计复杂的刺激界面。相比之下，Psychtoolbox能够实现大多数脑-机接口刺激界面，同时基于Matlab平台，集成了大量简单方便的函数，对科研人员的编程要求不高，基本上是科研人员的第一选择。

然而对我而言，我一直不喜欢Matlab，理由有三：
1. **Matlab不是一门真正的通用编程语言**。Matlab本质是为不懂CS的科研人员设计的编程语言，很难进行普通程序的开发，例如GUI、网络编程等等。脑机接口一个很重要的方面是开发和机器交互的程序，这些程序有时很注重性能，Matlab开发这些功能不太方便。
2. **Matlab是收费的商业软件**，其工具包的价格不是穷学生能承受的。尽管天朝存在“破解版”这种Matlab版本，MathWorks公司对科研人员使用破解版也视而不见，但谁知道以后会怎么样呢？为了不受制于人，我决定转向开源软件阵营。
3. **最重要的一点，学Matlab找不到工作**。研究生转博士期间，我也跟着校招参加了不少面试。很遗憾，在脑机接口领域，对口的工作几乎没有；就算扩大了从生物医学工程专业来说，大部分工作机会还是集中在医疗图像领域。这些领域的招聘要求中可没有熟练使用Matlab这一项，大部分还是C/C++、Java等通用编程语言。

综上所述，我在博士阶段毅然决然的放弃了Matlab，放弃了以前所有的代码，转向了Python。而在Python平台下，Psychopy几乎是唯一选择（个人认为）。Psychopy目前虽然仍处在开发阶段，还有不少bug，官方文档也不完善，但是官方社区和开发者相当活跃，使用人数也越来越多，借着Python语言的上升势头，我认为不久之后Psychopy很可能成为神经科学及脑机接口设计实验的首选框架。

当然由于Psychopy还很年轻，我在Psychopy实践过程中遇到许多问题。写Psychopy系列博文的目的是记录我的开发经验，给想使用Psyhcopy却遇到问题的朋友提供帮助。

## 安装Anaconda和Psychopy
Anaconda是一个用于科学计算的Python发行版，集成了大量Python科学计算所需的环境库，提供包管理和环境管理的功能，免去了手动安装Python及其各种工具包的麻烦。Anaconda支持Linux、Mac和Windows系统，在Windows下几乎是科学计算的唯一选择。我推荐安装Anaconda或Miniconda的最新版本，Anaconda的下载地址可在其[官网][1]或[清华TUNA镜像站][2]找到。

Psychopy在2019年10月8号release了3.2.4 (PyPi上为3.2.3)，相比3.0.0添加了很多特性，修复了大量bug。尽管官方提供了多种[安装方式][9]，我仍建议使用Anaconda或Miniconda安装（pip会存在一些编译依赖缺失问题）。

Psychopy在Python3.6版本下较为稳定，在Anaconda Prompt中键入如下命令创建环境并安装:
```
conda create -n psypy3 python=3.6
conda activate psypy3
conda install numpy scipy matplotlib pandas pyopengl pillow lxml openpyxl xlrd configobj pyyaml gevent greenlet msgpack-python psutil pytables requests[security] cffi seaborn wxpython cython pyzmq pyserial
conda install -c conda-forge pyglet pysoundfile python-bidi moviepy pyosf
pip install zmq json-tricks pyparallel sounddevice pygame pysoundcard psychopy_ext psychopy
```

在python控制台中运行如下命令检查Psychopy版本
```python
import psychopy
print(psychopy.__version__)
```

## 使用Psychopy
Psychopy提供两种刺激界面设计方式，一种是类似e-Prime的GUI界面Builder，另一种是普通的脚本编写方式Coder。
### Builder
在Anaconda Prompt中切换到Psychopy的app安装目录，Windows下通常为`cd %Anaconda的安装目录%\Anaconda\Lib\site-packages\psychopy\app`，在该目录下运行命令
```
python psychopyApp.py -b
```
![Builder界面]({% link /assets/img/builder.png %})
Builder的使用在[Builder - building experiments in a GUI][3]文档中有详细的介绍，builder很适合设计一些简单的刺激界面，设计完成的界面也可以转换为Coder中的脚本程序。

### Coder
在Anaconda Prompt中切换到Psychopy的app安装目录，Windows下通常为`%Anaconda的安装目录%\Anaconda\Lib\site-packages\psychopy\app`，在该目录下运行命令
```
python psychopyApp.py -c
```
![Coder界面]({% link /assets/img/coder.png %})
Coder的使用在[Coder - writing experiments with scripts][4]文档中有详细的介绍，相比Builder，Coder提供的编程设计方式更加灵活，可以实现更为复杂的刺激界面。当然Coder本身只是提供了开发环境，脚本编写可以在任何编辑器下进行，我很少直接使用Coder，通常会使用~~Pycharm和Sublime Text~~ vscode 来编写程序。

## Psychopy相关资源
~~Psychopy的官方文档更新不算及时，大部分文档还是基于Python2的版本~~Psychopy官方文档已更新至Python3，并不再支持Python2。官方文档和demo仍然是学习Psychopy的不二之选。如果有问题在官方文档里没有说明，Google也没有相关信息的话，可以去Psychopy的论坛问问或Github提个issue。

[Psychopy 官方文档][5]
[Psychopy API手册][6]
[Psychopy 论坛][7]
[Psychopy Github仓库][8]


[1]: https://www.anaconda.com/download/
[2]: https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/
[3]: http://www.psychopy.org/builder/builder.html
[4]: http://www.psychopy.org/coder/coder.html
[5]: https://discourse.psychopy.org/
[6]: http://www.psychopy.org/api/api.html
[7]: https://discourse.psychopy.org/
[8]: https://github.com/psychopy/psychopy
[9]: https://www.psychopy.org/download.html