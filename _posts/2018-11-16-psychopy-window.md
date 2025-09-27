---
title: 如何在Psychopy中新建窗口
date: 2018-11-16 21:15:22
updated: 2022-06-04 14:59:14
permalink: /psychopy-window/
categories: [psychopy,]
tags: [psychopy,]
toc: true
---

<!-- toc -->
窗口(windows)是刺激呈现的舞台，任何刺激对象都需要指定其所属的窗口对象。Pyschopy的Window对象位于psychopy.visual模块中，一个最简单的窗口示例如下<!--more-->

## 新建单窗口
```python
# -*- coding: utf-8 -*-

from psychopy import visual, event, monitors, core
import numpy as np

# 根据你自己的显示器调整显示器信息
mon = monitors.Monitor(
    name='my_monitor',
    width=53.704,  # 显示器宽度，单位cm
    distance=45,   # 被试距显示器距离，单位cm
    gamma=None,    # gamma值
    verbose=False) # 是否输出详细信息
mon.setSizePix((1920, 1080)) # 设置显示器分辨率
mon.save() # 保存显示器信息

win = visual.Window(monitor=mon, size=(800, 600), fullscr=False,
    screen=0, winType='pyglet', units='norm', allowGUI=True)

event.waitKeys() # 等待按键

# 修复或预防原始gamma不能恢复bug(运行Psychopy程序显示器变暗加入以下代码)
# Pyschopy 3.0.0 版似乎修复了此bug，如果显示器没有变暗的现象可以不加入以下代码
origLUT = np.round(win.backend._origGammaRamp * 65535.0).astype("uint16")
origLUT = origLUT.byteswap() / 255.0
win.backend._origGammaRamp = origLUT
core.quit() # 退出Psychopy程序
```

Window对象用size参数申明窗口尺寸为800\*600像素；fullscr参数决定是否全屏显示；screen参数决定了窗口在哪个显示器上显示，通常0是主显示器；winType参数决定了Psychopy使用的后端程序，有'pyglet'和'pygame'两种选择（Psychopy官方未来主要采用pyglet作为后端程序，我推荐采用pyglet）。
## 新建多窗口
Psychopy也可以同时建立多个窗口对象，注意仅pyglet后端支持多窗口行为。下面的代码展示了如何新建两个位于不同位置的窗口
```python
# -*- coding: utf-8 -*-

from psychopy import visual, event, monitors, core
import numpy as np

# 根据你自己的显示器调整显示器信息
mon = monitors.Monitor(
    name='my_monitor',
    width=53.704,  # 显示器宽度，单位cm
    distance=45,   # 被试距显示器距离，单位cm
    gamma=None,    # gamma值
    verbose=False) # 是否输出详细信息
mon.setSizePix((1920, 1080)) # 设置显示器分辨率
mon.save() # 保存显示器信息

# 窗口1在屏幕左上角
win1 = visual.Window(monitor=mon, size=(800, 600), pos=(0, 0), fullscr=False,
    screen=0, winType='pyglet', units='norm', allowGUI=True)

# 窗口2在屏幕右下角
win2 = visual.Window(monitor=mon, size=(800, 600), pos=(1120, 480), fullscr=False,
    screen=0, winType='pyglet', units='norm', allowGUI=True)

event.waitKeys() # 等待按键

# 修复或预防原始gamma不能恢复bug(运行Psychopy程序显示器变暗加入以下代码)
# Pyschopy 3.0.0 版似乎修复了此bug，如果显示器没有变暗的现象可以不加入以下代码

origLUT = np.round(win1.backend._origGammaRamp * 65535.0).astype("uint16")
origLUT = origLUT.byteswap() / 255.0
win1.backend._origGammaRamp = origLUT
win2.backend._origGammaRamp = origLUT
core.quit() # 退出Psychopy程序
```

pos参数调整窗口在屏幕上显示的位置，单位始终为像素，这里的坐标系不同于Psychopy的坐标系，以屏幕的左上角为原点，向下和向右分别为y轴和x轴的正方向。Window对象有很多可调整的参数和行为，具体细节可见[官方文档Window API][1]

[1]: http://www.psychopy.org/api/visual/window.html#psychopy.visual.Window
