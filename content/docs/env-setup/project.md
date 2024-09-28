---
title: "选择工程"
description: ""
summary: ""
date: 2024-03-09T05:21:17Z
lastmod: 2024-03-09T05:21:17Z
draft: false
weight: 204
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

我们提供了多个版本的工程，本文只列出可用的工程，以及如何下载。

我们会在下个章节中讨论编译及配置问题。

## 基于我们开发的版本

### 裸机

该版本完全基于官方 Pico C-SDK 开发，仅添加了LVGL的支持，所以如果您想要在本项目基础上进行原生二次开发，可以选择该裸机工程。

{{< callout context="note" title="说明" icon="info-circle" >}}
目前仅此工程兼容树莓派Pico 2 (RP2350)，其余工程正在开发中
若要为Pico 2 编译固件，在cmake配置时使用如下命令
```
# 位于build目录下
cmake -DPICO_BOARD=pico2 .. -G Ninja
```
{{< /callout >}}

国内用户
```shell
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_noos
```

```shell
git clone https://github.com/embeddedboys/pico_dm_qd3503728_noos
```

### FreeRTOS

与裸机版本不同的是，我们又在其上面添加了FreeRTOS的支持，同时该工程支持SMP，可同时使用RP2040的两个核心处理任务，如果您惯用FreeRTOS开发，可以选择本工程。

国内用户
```shell
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_freertos.git
```

```shell
git clone https://github.com/embeddedboys/pico_dm_qd3503728_freertos.git
```

### 独立于本项目的通用工程

在开发本项目的过程中，其实我们还同时开发着其他类似项目，为了加快后续适配工作进度，我们开发了本工程，通过简单的配置就可以在多个lcd或触摸之间切换，这意味着您完全可以使用本工程在您自己的平台上开发。 😎 有关本工程的详细内容参见其readme文件。

国内用户
```shell
git clone https://gitee.com/embeddedboys/pico_dm_8080_template
```

```shell
git clone https://github.com/embeddedboys/pico_dm_8080_template
```

#### 驱动支持情况

##### 显示驱动
- [x] ST6201
- [x] ST7789V
- [ ] ST7796U
- [x] ILI9806
- [x] ILI9488
- [ ] ILI9486
- [ ] ILI9341
- [x] R61581
- [x] 1P5623
- [x] LG4572B
- [ ] D51E5TA7601

##### 触摸驱动
- [x] FT6236U
- [x] NS2009
- [x] TSC2007
- [x] GT911

-----------------------------

### 如果上述版本都无法下载，尝试访问如下链接直接下载压缩包
链接：[https://pan.baidu.com/s/1m4WmPoHAZYiK3XwwXGrNDw?pwd=34mn](https://pan.baidu.com/s/1m4WmPoHAZYiK3XwwXGrNDw?pwd=34mn)

提取码：34mn


{{< callout context="note" title="说明" icon="info-circle" >}} 
该方式的源码版本可能比较落后，最新版本以github仓库为准。
我们也会及时更新镜像链接版本。
{{< /callout >}}

## 基于社区开源项目

 😋 我们正在开发中，包括但不限于如下工程：

- [x] Micropython
- [x] Arduino
- [ ] embedded_graphics (Rust)
- [ ] Slint (Rust)
- [ ] Nuttx
- [ ] zephyr

### Micropython

仓库链接：[https://github.com/embeddedboys/lv_micropython.git](https://github.com/embeddedboys/lv_micropython.git)

我们针对LVGL的V8.3和V9两个版本都做了适配，用户可根据需求自行选择合适的版本。

因为micropython工程涉及太多子模块，所以不方便迁移到gitee，除非在网络环境允许的情况下或您对micropython有源码修改需求，否则不建议用户自行编译。

您可以到[Github Release](https://github.com/embeddedboys/lv_micropython/releases)界面直接下载我们编译好的固件直接烧录使用，位于对应Release的Assets菜单下，在对应的Release中也有介绍使用发法。 烧录方法可以参考[固件烧录](/docs/get-started/固件烧录/)

对于无法访问Github的用户，可以访问如下链接下载固件：

[http://embeddedboys.com/uploads/qd3503728/micropython/](http://embeddedboys.com/uploads/qd3503728/micropython/)

为了帮助用户更快上手micropython开发，我们还录制了一个如何搭建micropython开发环境的视频，可通过访问如下链接查看：

（待添加）

#### 测试方法

先下载你需要的文件，如果您需要LVGL V8.3版本，则下载对应版本。

[http://embeddedboys.com/uploads/qd3503728/micropython/v8.3/](http://embeddedboys.com/uploads/qd3503728/micropython/v8.3/)

##### 1. 烧录固件到pico核心板

使用`firmware.uf2`或其他格式固件，按照适合您的方式烧录

##### 2. 使用Thonny或MicroPico上传库文件

可以点击如下链接下载Thonny IDE
[http://embeddedboys.com/uploads/qd3503728/micropython/thonny-4.1.4.exe](http://embeddedboys.com/uploads/qd3503728/micropython/thonny-4.1.4.exe)

Debian\Ubuntu用户可以执行如下命令安装Thonny IDE
```bash
bash <(wget -O - https://thonny.org/installer-for-linux)
```

安装好Thonny后，选择`工具`-->`选项`，切换到`解释器`选项卡，在`Thonny应该使用哪种解释器来运行您的代码？`下拉菜单中找到`MicroPython (Raspberry Pi Pico)`，并在`端口或WebREPL`下拉菜单选择`<自动探测端口>`，然后取消勾选`运行代码前，先重启解释器`选项。

可以使用Thonny或者MicroPico的内置功能，保存`lv_utils.py`到核心板文件系统中。 以Thonny为例，打开`lv_utils.py`文件后，选择`文件`-->`另存为`，在弹出的对话框中选择`Raspberry Pi Pico`，随后在弹出的**文件浏览器**的**文件名输入框**中输入`lv_utils.py`，然后点击确定保存。

{{< details "lv_utils.py">}}
```bash {title="v8.3"}
##############################################################################
# Event Loop module: advancing tick count and scheduling lvgl task handler.
# Import after lvgl module.
# This should be imported and used by display driver.
# Display driver should first check if an event loop is already running.
#
# Usage example with SDL:
#
#        SDL.init(auto_refresh=False)
#        # Register SDL display driver.
#        # Regsiter SDL mouse driver
#        event_loop = lv_utils.event_loop()
#
#
# uasyncio example with SDL:
#
#        SDL.init(auto_refresh=False)
#        # Register SDL display driver.
#        # Regsiter SDL mouse driver
#        event_loop = lv_utils.event_loop(asynchronous=True)
#        uasyncio.Loop.run_forever()
#
# uasyncio example with ili9341:
#
#        event_loop = lv_utils.event_loop(asynchronous=True) # Optional!
#        self.disp = ili9341(asynchronous=True)
#        uasyncio.Loop.run_forever()
#
# MIT license; Copyright (c) 2021 Amir Gonnen
#
##############################################################################

import lvgl as lv
import micropython
import usys

# Try standard machine.Timer, or custom timer from lv_timer, if available

try:
    from machine import Timer
except:
    try:
        from lv_timer import Timer
    except:
        raise RuntimeError("Missing machine.Timer implementation!")

# Try to determine default timer id

default_timer_id = 0
if usys.platform == 'pyboard':
    # stm32 only supports SW timer -1
    default_timer_id = -1
    
if usys.platform == 'rp2':
    # rp2 only supports SW timer -1
    default_timer_id = -1

# Try importing uasyncio, if available

try:
    import uasyncio
    uasyncio_available = True
except:
    uasyncio_available = False

##############################################################################

class event_loop():

    _current_instance = None

    def __init__(self, freq=25, timer_id=default_timer_id, max_scheduled=2, refresh_cb=None, asynchronous=False, exception_sink=None):
        if self.is_running():
            raise RuntimeError("Event loop is already running!")

        if not lv.is_initialized():
            lv.init()

        event_loop._current_instance = self

        self.delay = 1000 // freq
        self.refresh_cb = refresh_cb
        self.exception_sink = exception_sink if exception_sink else self.default_exception_sink

        self.asynchronous = asynchronous
        if self.asynchronous:
            if not uasyncio_available:
                raise RuntimeError("Cannot run asynchronous event loop. uasyncio is not available!")
            self.refresh_event = uasyncio.Event()
            self.refresh_task = uasyncio.create_task(self.async_refresh())
            self.timer_task = uasyncio.create_task(self.async_timer())
        else:
            self.timer = Timer(timer_id)
            self.task_handler_ref = self.task_handler  # Allocation occurs here
            self.timer.init(mode=Timer.PERIODIC, period=self.delay, callback=self.timer_cb)
            self.max_scheduled = max_scheduled
            self.scheduled = 0

    def deinit(self):
        if self.asynchronous:
            self.refresh_task.cancel()
            self.timer_task.cancel()
        else:
            self.timer.deinit()
        event_loop._current_instance = None

    def disable(self):
        self.scheduled += self.max_scheduled

    def enable(self):
        self.scheduled -= self.max_scheduled

    @staticmethod
    def is_running():
        return event_loop._current_instance is not None

    @staticmethod
    def current_instance():
        return event_loop._current_instance

    def task_handler(self, _):
        try:
            lv.task_handler()
            if self.refresh_cb: self.refresh_cb()
            self.scheduled -= 1
        except Exception as e:
            if self.exception_sink:
                self.exception_sink(e)

    def timer_cb(self, t):
        # Can be called in Interrupt context
        # Use task_handler_ref since passing self.task_handler would cause allocation.
        lv.tick_inc(self.delay)
        if self.scheduled < self.max_scheduled:
            try:
                micropython.schedule(self.task_handler_ref, 0)
                self.scheduled += 1
            except:
                pass

    async def async_refresh(self):
        while True:
            await self.refresh_event.wait()
            self.refresh_event.clear()
            try:
                lv.task_handler()
            except Exception as e:
                if self.exception_sink:
                    self.exception_sink(e)
            if self.refresh_cb: self.refresh_cb()

    async def async_timer(self):
        while True:
            await uasyncio.sleep_ms(self.delay)
            lv.tick_inc(self.delay)
            self.refresh_event.set()
            

    def default_exception_sink(self, e):
        usys.print_exception(e)
        event_loop.current_instance().deinit()
```
{{< /details >}}

##### 3. 运行测试

执行`ili9488_test.py`，此时屏幕应有“Hello World”按钮出现，且点击按钮应有反馈。

{{< details "ili9488_test.py">}}
```bash {title="v8.3"}
# init
import machine

import usys as sys
sys.path.append('') # See: https://github.com/micropython/micropython/issues/6419

import lvgl as lv
import lv_utils

lv.init()

class driver:
    def __init__(self):
        machine.freq(240000000)  # set the CPU frequency to 240 MHz
        print("CPU freq : ", machine.freq() / 1000000, "MHz")

    def init_gui(self):
        import ili9488 as tft
        import ft6236 as tp

        hres = 480
        vres = 320

        # Register display driver
        event_loop = lv_utils.event_loop()
        tft.deinit()
        tft.init()
        tp.init()

        disp_buf1 = lv.disp_draw_buf_t()
        buf1_1 = tft.framebuffer(1)
        buf1_2 = tft.framebuffer(2)
        disp_buf1.init(buf1_1, buf1_2, len(buf1_1) // lv.color_t.__SIZE__)
        disp_drv = lv.disp_drv_t()
        disp_drv.init()
        disp_drv.draw_buf = disp_buf1
        disp_drv.flush_cb = tft.flush
        # disp_drv.gpu_blend_cb = tft.gpu_blend
        # disp_drv.gpu_fill_cb = tft.gpu_fill
        disp_drv.hor_res = hres
        disp_drv.ver_res = vres
        disp_drv.register()

        # Register touch sensor
        indev_drv = lv.indev_drv_t()
        indev_drv.init()
        indev_drv.type = lv.INDEV_TYPE.POINTER
        indev_drv.read_cb = tp.ts_read
        indev_drv.register()

if not lv_utils.event_loop.is_running():
    drv = driver()
    drv.init_gui()

############################################################################################

scr = lv.obj()
btn = lv.btn(scr)
btn.align(lv.ALIGN.CENTER, 0, 0)
label = lv.label(btn)
label.set_text('Hello World!')
lv.scr_load(scr)
```
{{< /details >}}

您也可以使用在线模拟器来调试micropython程序

V8.3: [https://sim.lvgl.io/v8.3/micropython/ports/javascript/index.html](https://sim.lvgl.io/v8.3/micropython/ports/javascript/index.html)

V9.0: [https://sim.lvgl.io/v9.0/micropython/ports/webassembly/index.html](https://sim.lvgl.io/v9.0/micropython/ports/webassembly/index.html)

可在[lv_mpy_examples_v8](https://github.com/uraich/lv_mpy_examples_v8)这个仓库查看v8.3版本的lvgl micropython例子，因为v9版本较新，暂时没有例程参考。

### Arduino

我们已经添加了一个初步支持的Arduino移植，可查看如下仓库
[https://github.com/embeddedboys/pico_dm_qd3503728_arduino](https://github.com/embeddedboys/pico_dm_qd3503728_arduino)

这部分的文档还在整理中，可先查看如上仓库的 readme 简易说明

### embedded_graphics (Rust)

[https://github.com/embedded-graphics/embedded-graphics](https://github.com/embedded-graphics/embedded-graphics)

`embedded_graphics` 是一款专注于内存受限的嵌入式设备的二维图形库，基于Rust开发。

底层刷图驱动支持情况：
- [x] GPIO
- [ ] PIO
- [ ] PIO + DMA

仓库链接：[https://github.com/embeddedboys/pico_dm_qd3503728_embedded_graphics](https://github.com/embeddedboys/pico_dm_qd3503728_embedded_graphics)

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_embedded_graphics
```

编译烧录参考[编译及配置](../编译及配置/#embedded_graphics)

### Slint (Rust)

[https://slint.dev/](https://slint.dev/)

已提上日程，开发中。。。

### Nuttx

[https://nuttx.apache.org/docs/latest/platforms/arm/rp2040/index.html](https://nuttx.apache.org/docs/latest/platforms/arm/rp2040/index.html)

已提上日程，调研中。。。

### Zephyr

[https://docs.zephyrproject.org/latest/boards/raspberrypi/rpi_pico/doc/index.html](https://docs.zephyrproject.org/latest/boards/raspberrypi/rpi_pico/doc/index.html)

已提上日程，调研中。。。