---
title: Makefile使用规则
categories: [c++]
tags: [c++, makefile]
permalink: /learn-makefile/
date: 2022-06-07 00:00:00
---

<!-- toc -->

出于工作需要，我要开始系统学习c++了。目前我的主力台式机是Linux系统，最常用的编辑器是VS Code，所以想要得到一个比较完整的C/C++工程方案，似乎学习Makefile的相关规则是必不可少的。本文的内容主要源于[makefile tutorial by example](https://makefiletutorial.com/)。<!--more-->

### 为什么要使用makefile

对于大型C/C++项目而言，完整的程序功能往往是很多子模块组合得到的。试想开发者改变或添加了几个新的功能，如果手动重新编译整个项目（比如在shell里输入一长串的g++命令以及一堆的lib），不仅费时费力、还容易出错。make工具正是用来处理这种某些模型源代码文件需要重新编译的情况，它所依赖的配置文件就是Makefile。简而言之，make通过预先配置好的Makefile，按一定的规则负责处理C/C++项目的编译，从而得到最终的可执行程序。目前，大多数开源项目都会提供Makefile，开发者可以很方便的调用`make`命令编译整个开源项目。

当然，make并不是唯一用于处理C/C++项目编译的解决方案，一些替代的解决方案包括[CMake](https://cmake.org)、[Bazel](https://bazel.build/)等等。在Windows平台上，Visual Studio也有其内置的build工具。

本篇所使用的make是指GNU make，在Linux和MacOS上是默认安装的make实现。

### 基本语法

现在我们来创建一个最简单的Makefile。首先在任何目录下创建一个名为`Makefile`的文件（注意文件名就是Makefile，大些的M，也没有类似.txt的后缀），然后在其中输入这些文本：
```Makefile
hello:
	echo "hello make"
```

注意，Makefile的缩进只能是Tab，不可以用空格，不然`make`会报缺失分隔符错误，要小心你的编辑器是不是自动将Tab缩进转换成了4个空格。接下来在该目录下的terminal里运行`make`命令，输出如下：
```shell
$ make
echo "hello make"
hello make
```
表明需要执行`echo "hello make"`命令，执行结果是`hello make`。

Makefile文件是由一系列rule组成的，一个rule定义如下：
```Makefile
targets: prerequisties
    command
    command
    command
```

`targets`是本次操作希望达成的目标，它实质是一系列文件名，相互之间用空格隔开，通常只有一个文件名，比如例子中的`hello`文件，而每一行的`command`则是用来达成`targets`的手段，通过执行`command`，我们希望在所有命令执行完成后能够产生`targets`所规定的文件。`prerequisties`同样也是一系列文件名，相互之间用空格隔开，它表示执行命令前这些文件应当已经存在了，有些类似程序的依赖概念。一个rule可以简单理解为在`prerequisties`存在的情况下，运行一系列`command`，希望最终能够得到`targets`中的文件。

### 基本执行关系

```Makefile
blah: blah.o
	gcc blah.o -o blah # Runs third

blah.o: blah.c
	gcc -c blah.c -o blah.o # Runs second

blah.c:
	echo "int main() { return 0; }" > blah.c # Runs first

clean:
	rm -f blah blah.o blah.c
```

上面是一个有四个rule的Makefile文件，不指定target情况下，运行`make`会默认以第一个rule中的targets为目标，从而进行如下操作：
- 第一个rule想产生名为`blah`的文件，但是需要文件`blah.o`，所以跳转到第二个rule
- 第二个rule又需要文件`blah.c`，所以跳转到第三个rule
- 第三个rule为了产生`blah.c`，执行命令产生一个blah.c文件，接着返回第二个rule
- 现在有了`blah.c`，第二个rule编译生成`blah.o`二进制文件，接着返回第一个rule
- 现在有了`blah.o`，第一个rule链接生成`blah`可执行文件，任务完成

现在我们的目录下已经有`blah`文件了，所以再运行一次`make`不会产生任何效果。注意到`blah`目标的产生与`clean`无关，所以`clean`所代表的rule不会执行，不过可以显式运行`make clean`命令来调用该rule规定的清理文件操作。

最后需要注意的是，`make`似乎只会检查一次`targets`和`prerequisties`是否存在，至于调用之后是不是成功产生了目标文件，`make`是不会去检查的，而是默认成功了。