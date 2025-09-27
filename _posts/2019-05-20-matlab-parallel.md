---
title: MATLAB分布式集群搭建记录
categories: [matlab,]
tags: [matlab,]
permalink: /matlab-mdce-log/
date: 2019-05-20 00:00:00
updated: 2022-03-21 00:00:00
toc: true
---

<!-- toc -->

**本篇的内容可能过时啦**

虽然我很久不用MATLAB处理日常工作，但是实验室主流依然是MATLAB（用Python的就那么几个T_T)。以前小伙伴们跑程序都是拷贝程序和数据到实验室的计算服务器上，手工开N个MATLAB窗口做运算。现在实验室规模扩大，这种手工的方式越来越繁琐。我从前用MATLAB时就想试试集群计算，奈何当时实验室没啥硬件条件，正好现在有机会，我干脆搭了个MATLAB集群供小伙伴使用。<!--more-->

## 软硬件
硬件方面：
 - 4核心, 16GB内存， 百兆网卡普通台式机(manage节点)
 - 40核心, 128GB内存, 千兆网卡计算服务器(compute1节点)
 - 346TB存储， 千兆网卡存储服务器(storage节点)

软件方面：
 - Windows10专业版系统
 - centos7
 - matlab2017b
 
网络环境：
- 192.168.130.12(matlab-manage.xxx.org) -- manage节点
- 192.168.130.11(matlab-compute1.xxx.org) -- compute1节点
- 192.168.130.10 -- storage节点

MATLAB的帮助文档中提出，想要使用集群计算服务应该满足以下条件：
 - 推荐一个CPU核心最多创建一个worker
 - 推荐每个worker最少可以使用2GB内存
 - 最少5GB的硬盘空间容纳暂时性的数据文件
 - 计算集群之间应当使用同构的计算架构(要求计算节点的硬件配置、系统和软件配置一致)

## 集群安装配置

### ip域名设置
修改compute节点和manage节点的计算机名、ip地址以及DNS域名解析，例如compute1节点的计算机名为matlab-compute1.xxx.org(xxx.org为后缀域名)，DNS域名也应该为matlab-compute1.xxx.org，ip地址为192.168.130.11。

>MATLAB分布式计算服务似乎要求计算机名要添加后缀域名(xxx.org)，否则在集群测试时会有解析不匹配的警告，Windows专业版可在**这台电脑-属性-更改设置-更改-其他**中添加主DNS后缀。

### manage节点
在manage节点安装matlab2017b，manage节点在安装过程中应该勾选MATLAB License Server和MATLAB Distributed Computing Server工具箱，前者为集群提供license认证服务，后者是分布式计算的核心服务组件。对于破解版的MATLAB，应该输入floating license的key而不是standalone的key，才能安装MATLAB License Server。安装完毕（并破解）后，在Windows服务选项卡中启动MATLAB License Server服务。
![license]({% link /assets/img/license.png %})

同时修改C:\Program Files\MATLAB\R2017b\licenses\network.lic为如下内容
```
SERVER this_host ANY
USE_SERVER
```
修改C:\Program Files\MATLAB\R2017b\toolbox\distcomp\bin\mdce_def.bat其中的security level为2
```
set SECURITY_LEVEL=2
```

>设置security level为2的效果是要求用户在使用分布式计算服务时输入用户名，从而可以监控集群使用情况。

启动MATLAB，切换到C:\Program Files\MATLAB\R2017b\toolbox\distcomp\bin目录下，在MATLAB命令行窗口输入如下命令安装并启动mdce服务
```
!mdce install 
!mdce start
```

>最好在MATLAB命令行窗口内启动mdce服务，如果在Windows服务选项卡中启动服务，会出现权限问题导致集群worker无法连接。

启动mdce服务后最好双击运行C:\Program Files\MATLAB\R2017b\toolbox\distcomp\bin\addMatlabToWindowsFirewall.bat文件（我的做法是直接关闭Windows防火墙避免多余的问题）

### compute节点
compute节点的安装配置同manage节点，仅以下内容不同
1. 安装matlab时无需勾选MATLAB License Server工具箱
2. 修改C:\Program Files\MATLAB\R2017b\licenses\network.lic为如下内容
```
SERVER matlab-manage.xxx.org ANY
USE_SERVER
```
此外为了跟storage节点连接，compute节点需要安装NFS服务，在**程序和功能-启用或关闭Windows功能**中勾选NFS服务
![nfs]({% link /assets/img/nfs.png %})

## storage节点
storage节点设置NFS服务，NFS服务端安装和配置网上都有，我就不写了。

### 添加集群节点
在manage节点运行C:\Program Files\MATLAB\R2017b\toolbox\distcomp\bin\admincenter.bat，启动管理面板，点击**Add or Find**，添加manage节点和compute1节点，添加完毕后，点击**Test Connectivity**，测试通过如下图
![connectivity]({% link /assets/img/connectivity.png %})

在MATLAB Job Scheduler面板点击start启动scheduler，输入名称，选择scheduler的节点为matlab-manage.xxx.org，因为security level为2，还需要设置管理员的密码。

设置好scheduler后，右键scheduler点击Start Workers，勾选compute1节点，设置启动的worker数量（我只有40个核心，所以启动40个worker）。
![center]({% link /assets/img/center.png %})

### 客户端配置和使用
MATLAB集群计算要求客户端的matlab版本和服务端一致，因为我服务端安装的是2017b，客户端也应该是matlab2017b。客户端可以选择standalone安装方式，也需要安装mdce服务添加防火墙配置并启动。
如果客户端想直接使用NFS服务，也需要在**程序和功能-启用或关闭Windows功能**中勾选NFS服务。
安装完毕后，在MATLAB主页中的**Parallel**选项选择**Discover Cluster**，勾选**On your network**，点击Next等待发现集群mjs40_2，选择集群，点击Next，Finish，就可以使用集群了，集群的使用情况可以在**Parallel**选项里**Monitor Jobs**查看。

这里提供两个matlab并行计算脚本检测集群配置是否正确
```matlab
%This demo shows how to use distributed computing server

primeNumbers = primes(uint64(2^21));
compositeNumbers = primeNumbers.*primeNumbers(randperm(numel(primeNumbers)));
factors = zeros(numel(primeNumbers),2);
tic;
parfor idx = 1:numel(compositeNumbers)
    factors(idx,:) = factor(compositeNumbers(idx));
end
toc
```
```matlab
%This demo shows how to load data from nfs server, target_folder is nfs server ip address

target_folder='\\192.168.130.10\pub\data\';
factors=zeros(400,2);
tic;
parfor i=0:399
    tmp = load([target_folder,num2str(i),'.mat']);
    data = tmp.data;
    factors(i+1, :)=factor(data);
end
toc
```