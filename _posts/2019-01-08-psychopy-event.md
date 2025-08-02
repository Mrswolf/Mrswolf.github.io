---
title: Psychopy事件响应
date: 2019-01-08 17:01:30
updated: 2022-06-04 14:56:24
permalink: psychopy-event
categories: [psychopy,]
tags: [psychopy,]
---
<!-- toc -->

## Psychopy事件响应
Psychopy提供了很多IO交互方式，当然，最根本的还是键盘和鼠标。本节介绍Psychopy鼠标和键盘的编程技巧。<!-- more -->

### 全局按键响应
编写刺激界面免不了要反复调试，要看看字体颜色对不对、图形大小合不合适，一旦发现刺激界面需要改进就得退出程序修改源代码。

如果采用普通的按键检测方式，则需要在一个循环体内检查按键状态，这显然有可能造成不可知的错误（比如在检测按键前进入一个死循环函数，程序永远无法退出啦），这个时候全局按键响应就很有用了。

Psychopy用[psychopy.event.globalkeys][1]来设置全局按键，官方文档里没有如何使用全局按键的说明，但在coder的Demo里有演示global_event_keys.py。

global_event.keys.py程序注册了三个按键，按键“b”调用python的setattr函数，设置rect对象的填充颜色为蓝色,按键“ctrl”+“r”调用python的setattr函数，设置rect对象的填充颜色为红色。按键“q”调用core.quit方法终止程序退出。
```python
# -*- coding: utf-8 -*-

from psychopy import core, event, visual, monitors


if __name__=='__main__':
    mon = monitors.Monitor(
        name='my_monitor',
        width=53.704,  # 显示器宽度，单位cm
        distance=45,   # 被试距显示器距离，单位cm
        gamma=None,    # gamma值
        verbose=False) # 是否输出详细信息
    mon.setSizePix((1920, 1080)) # 设置显示器分辨率
    mon.save() # 保存显示器信息

    win = visual.Window(monitor=mon, size=(800, 600), fullscr=False,
        screen=0, winType='pyglet', units='norm', allowGUI=False)
    rect = visual.Rect(win, fillColor='blue', pos=(0, -0.2))
    text = visual.TextStim(
        win,
        pos=(0, 0.5),
        text=('Press\n\n'
              'B for blue rectangle,\n'
              'CTRL + R for red rectangle,\n'
              'Q or ESC to quit.'))

    # Add an event key.
    event.globalKeys.add(key='b', func=setattr,
                         func_args=(rect, 'fillColor', 'blue'),
                         name='blue rect')

    # Add an event key with a "modifier" (CTRL).
    event.globalKeys.add(key='r', modifiers=['ctrl'], func=setattr,
                         func_args=(rect, 'fillColor', 'red'),
                         name='red rect')

    # Add multiple shutdown keys "at once".
    for key in ['q', 'escape']:
        event.globalKeys.add(key, func=core.quit)

    # Print all currently defined global event keys.
    print(event.globalKeys)
    print(repr(event.globalKeys))

    while True:
        text.draw()
        rect.draw()
        win.flip()
```

以下是event.globalkeys.add()方法的参数介绍
event.globalkeys.add(key, func, func_args=(), func_kwargs=None, modifiers=(), name=None)

|parameters|type|description|
|:---------|:---|:----------|
|key|string|按键字符串|
|func|function|按键时执行的函数|
|func_args|iterable|函数的args参数|
|func_kwargs|dict|函数的kwargs参数|
|modifiers|iterable|组合按键字符串列表，例如'shift','ctrl','alt','capslock','scrollock'等|
|name|string|按键事件的名称|

此外还有event.globalkeys.remove()方法以移除全局按键
event.globalkeys.remove(key, modifiers=())

|parameters|type|description|
|:---------|:---|:----------|
|key|string|按键字符串|
|modifiers|iterable|组合按键字符串列表，例如'shift','ctrl','alt','capslock','scrollock'等|

### 等待按键和检测按键
除了全局按键响应，Psychopy还提供了等待按键响应和检测按键响应两种方式。

以下为等待按键函数event.waitKeys()的演示程序，按'esc'或五次其他按键退出程序。
```python
# -*- coding: utf-8 -*-

from psychopy import core, event, visual, monitors


if __name__=='__main__':
    mon = monitors.Monitor(
        name='my_monitor',
        width=53.704,  # 显示器宽度，单位cm
        distance=45,   # 被试距显示器距离，单位cm
        gamma=None,    # gamma值
        verbose=False) # 是否输出详细信息
    mon.setSizePix((1920, 1080)) # 设置显示器分辨率
    mon.save() # 保存显示器信息

    win = visual.Window(monitor=mon, size=(800, 600), fullscr=False,
        screen=0, winType='pyglet', units='norm', allowGUI=False)
    msg = visual.TextStim(win, text='press a key\n < esc > to quit')
    msg.draw()
    win.flip()

    k = ['']
    count = 0
    while k[0] not in ['escape', 'esc'] and count < 5:
        k = event.waitKeys()
        print(k)
        count += 1

    win.close()
    core.quit()
```

event.waitKeys()阻塞函数进程直到被试按键，以下是[event.waitKeys()][2]方法的参数介绍
event.waitKeys(maxWait=inf, keyList=None, modifiers=False, timeStamped=False, clearEvents=True)

|parameters|type|description|
|:---------|:---|:----------|
|maxWait|numeric value|最大等待时间，默认为inf|
|keyList|iterable|指定函数检测的按键名称，函数仅在按指定键时返回|
|modifiers|bool|如果True，返回(keyname, modifiers)的tuple|
|timeStamped|bool|如果True，返回(keyname, time)|
|clearEvents|bool|如果True，在检测新的按键前清理event buffer|

|return|type|description|
|:-----|:---|:----------|
|keys|iterable|按键列表；超时返回None|

等待按键会阻塞进程，Psychopy还提供了另一种非阻塞检测方式[event.getKeys()][2]。

以下代码如下不断检测按键并输出，直到按'escape'键退出程序。
```python
# -*- coding: utf-8 -*-

from psychopy import core, event, visual, monitors


if __name__=='__main__':
    mon = monitors.Monitor(
        name='my_monitor',
        width=53.704,  # 显示器宽度，单位cm
        distance=45,   # 被试距显示器距离，单位cm
        gamma=None,    # gamma值
        verbose=False) # 是否输出详细信息
    mon.setSizePix((1920, 1080)) # 设置显示器分辨率
    mon.save() # 保存显示器信息

    win = visual.Window(monitor=mon, size=(800, 600), fullscr=False,
        screen=0, winType='pyglet', units='norm', allowGUI=False)
    msg = visual.TextStim(win, text='press a key\n < esc > to quit')
    msg.draw()
    win.flip()

    count = 0
    while True:
        k = event.getKeys()
        if k:
            if 'escape' in k:
                break
            print(k)

    win.close()
    core.quit()
```

以下是[event.getKeys()][2]方法的参数介绍
event.getKeys(keyList=None, modifiers=False, timeStamped=False)

|parameters|type|description|
|:---------|:---|:----------|
|keyList|iterable|指定函数检测的按键名称，函数仅在按指定键时返回|
|modifiers|bool|如果True，返回(keyname, modifiers)的tuple|
|timeStamped|bool|如果True，返回(keyname, time)|

|return|type|description|
|:-----|:---|:----------|
|keys|iterable|按键列表；超时返回None|

### 鼠标事件
Psychopy提供[event.Mouse][2]类来处理鼠标相关的事件，官方文档对此有详细的介绍。以下的代码显示了一个含有矩形的窗，在矩形内部单击左右键可以改变颜色，而按中央滚轮键则退出程序。
```python
# -*- coding: utf-8 -*-

from psychopy import core, event, visual, monitors


if __name__=='__main__':
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
    rect = visual.Rect(win, fillColor='blue', pos=(0, 0))
    
    # 创建Mouse类
    mouse = event.Mouse(visible=True, newPos=(0, 0), win=win)
    while True:
        # 重置单击事件状态
        mouse.clickReset()

        # 检测左键是否在矩形内单击
        if mouse.isPressedIn(rect, buttons=[0]):
            rect.fillColor = 'red'

        # 检测右键是否在矩形内单击
        if mouse.isPressedIn(rect, buttons=[2]):
            rect.fillColor = 'blue'
        
        # 检测是否单击滚轮键
        # button1: left click
        # button2: middle click
        # button3: right click
        button1, button2, button3 = mouse.getPressed(getTime=False)
        if button2:
            break
        
        rect.draw()
        win.flip()

    core.quit()
```


[1]: https://github.com/psychopy/psychopy/blob/master/psychopy/event.py
[2]: http://www.psychopy.org/api/event.html