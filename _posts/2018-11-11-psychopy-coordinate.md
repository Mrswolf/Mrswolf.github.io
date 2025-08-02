---
title: Psychopy坐标系统与显示器设置
date: 2018-11-11 21:20:16
updated: 2022-06-04 14:49:00
permalink: /psychopy-coordinate
categories: [psychopy,]
tags: [psychopy,]
---

<!-- toc -->

## Psychopy的坐标系统
Psychopy提供了5种不同的坐标单位(unit)，使用者只需提供刺激对应的坐标单位，Psychopy会自动计算刺激所对应的像素点范围。这种多坐标单位的好处在于，能够开发和设备无关的刺激呈现，不需要每次实验都对刺激的大小和呈现位置进行调整。其劣势则是需要精心挑选刺激对应的坐标单位，有时还要在不同单位间进行转换，一不小心就容易出错。<!-- more -->

无论何种单位，Psychopy的坐标系统始终以屏幕中心为原点(0, 0)，原点向上为正y轴，原点向右为正x轴。不同的Psychopy坐标单位的所需参考对象不同，例如norm、height的坐标单位是针对视窗对象(window)，而cm、deg、degFlat、degFlatPos、pix则是针对屏幕(screen)。
![坐标系统]({% link /assets/img/coordinate.png %})

### 归一化单位(norm)
归一化单位可能是最常用的单位之一。在该单位下，window左下角坐标为(-1, -1)，window右上角坐标为(1, 1)。如图为长宽均为0.5的三个色块，其中心点分别位于(-0,5, 0)、(0, 0)和(0.5, 0)坐标下，注意window的分辨率为800\*600，因此尽管色块的归一化长宽均为0.5，但其长实际为200像素点，宽实际为150像素点，表现为长方形。
![norm单位]({% link /assets/img/norm.png %})

### 像素单位(pix)
像素单位的坐标范围取决于screen的宽、高像素点数，假设screen宽度有w个像素点，高度有h个像素点，则screen左下角坐标为(-w/2, -h/2)，右上角坐标为(w/2, h/2)。

### 厘米单位(cm)
厘米单位的坐标范围取决于screen的宽度和高度，假设screen宽度为w厘米，高度为h厘米，则screen左下角坐标为(-w/2, -h/2)，右上角坐标为(w/2, h/2)，每cm所代表的像素长度则由screen的像素点数确定。

### 高度单位(height)
高度单位的坐标范围取决于window的宽高比。无论何种window，y轴的坐标范围始终是从-0.5到0.5。因此，如果window是4：3的尺寸，则window左下角坐标为(-0.6667, -0.5)，window右上角坐标为(0.6667, 0.5)；如果是16：9的尺寸，则window左下角坐标为(-0.8, -0.5)，window右上角坐标为(0.8, 0.5)。如图是800\*600的window，色块的长宽均为1，则色块会占满y轴方向的所有空间。
![height单位]({% link /assets/img/height.png %})

### 视角(deg, degFlatPos, degFlat)
视角单位是五种单位种最复杂的坐标单位，使用该单位，不仅要知道屏幕的大小、像素点的多少，还要知道被试距离屏幕的距离，Psychopy提供三种不同的视角单位deg、degFlatPos和degFlat。
![视角单位]({% link /assets/img/deg.png %})

#### deg
deg单位默认视角在screen所有位置具有相同的像素长度，即在screen边缘位置和中心位置会产生相同大小的刺激图形。采用deg单位可以认为screen是球形曲面，而人眼则是球心，每度视角在screen所投射的像素长度完全相同。上图deg行红绿蓝三色块的长宽均为5度，位置分别为(-25, 10)、(0, 10)、(25, 10)。

#### degFlatPos
degFlatPos在deg的基础上考虑了位置在水平屏幕上的修正，远离屏幕中心的位置，刺激间的间隔越大，但是不改变刺激本身的大小。上图degFlatPos行三色块的参数同deg行完全相同，但因为采用了degFlatPos单位，红蓝色块距离绿色色块的距离要比deg行更大。

#### degFlat
degFlat不仅修正了位置信息，还修正了刺激的大小，因此，远离屏幕中心的位置，刺激尺寸越大，刺激间的间隔也越大。上图degFlat行三色块的参数同deg行完全相同，但因为采用了degFlat单位，不仅红蓝色块距离绿色色块的距离要比deg行更大，红蓝色块的形状也产生了畸变。

### Psychopy单位转换
Psychopy提供了不同单位间的转换方法，位于[psychopy.tools.unittools][1]模块中。

## Psychopy显示器信息设置

以上提到的pix、cm、deg、degFlatPos和degFlag均需要提供显示器信息（尺寸、分辨率等），Psychopy提供两种方式设定显示设备信息。

### Moniter Center界面
Anaconda Prompt下切换到`%Anaconda的安装目录%\Anaconda\Lib\site-packages\psychopy\monitors`目录，输入命令
```
python MonitorCenter.py
```
![Moniter Center界面]({% link /assets/img/monitor_center.png %})

### Moniter类
Moniter类位于[psychopy.monitors][2]模块中，负责显示器参数和刺激环境设置。
```python
# Create my primary monitor
mon = monitors.Monitor(
    'monitor1',
    width=53.704,  # width of the monitor in cm
    distance=114,  # distance from viewer to the screen in cm
    notes="This is my primary monitor")

mon.setSizePix((1920, 1080))  # set pixel size of the monitor
mon.save()  # save the monitor information to disk

# Reuse my primary monitor
mon = monitors.Monitor('monitor1')
# Change the distance from viewer to the screen
mon.setDistance(200)
```
Monitor.save()函数保存的显示器信息位于`%APPDATA%psychopy3monitors`文件夹下，保存过一次后可以直接在Monitor类中以name调用。

[1]: https://www.psychopy.org/api/tools/unittools.html
[2]: http://www.psychopy.org/api/monitors.html