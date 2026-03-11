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

如果您对这些工程的移植过程感兴趣，可以参考[移植教程](../../porting/)中的对应内容。

ESP32，Linux平台因为篇幅过长，所以放到了单独的章节中。

我们会在下个章节中讨论工程的编译及配置问题。

## 基于我们开发的版本

😋 我们正在开发中，包括但不限于如下工程：

- [x] [裸机](#裸机)
- [x] [USB 显示屏（开发中）](#usb-display)
- [x] [RP2350 LVGL全屏刷新示例](#rp2350-lvgl全屏刷新示例)
- [x] [EEZ Studio示例工程](#eez-studio-lvgl-示例工程)
- [x] [8080屏模板工程](#8080屏模板工程)
- [x] [FreeRTOS](#freertos)
- [x] [ESP32](#esp32-s3)
- [x] [Linux](#linux)
- [x] [Micropython (Python)](#micropython-python)
- [x] [Arduino](#arduino)
- [x] [embedded_graphics (Rust)](#embedded_graphics-rust)
- [x] [Slint (Rust)](#slint-rust)
- [x] [zephyr](#zephyr)
- [ ] [Nuttx](#nuttx)
- [x] [hagl](#hagl)
- [x] [AWTK](#awtk)
- [x] [uMac](#umac)
- [x] [SGL](#sgl)
- [x] [uGUI](#ugui)

### 裸机

该版本完全基于官方 Pico C-SDK 开发，仅添加了LVGL的支持，所以如果您想要在本项目基础上进行原生二次开发，可以选择该裸机工程。

仓库链接：[https://gitee.com/embeddedboys/pico_dm_qd3503728_noos](https://gitee.com/embeddedboys/pico_dm_qd3503728_noos)

国内用户：

```shell
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_noos
```

国外用户：

```shell
git clone https://github.com/embeddedboys/pico_dm_qd3503728_noos
```

### USB Display

我们将在现有工程基础上（裸机或者FreeRTOS工程），添加 USB 显示屏 的支持，这将达到如下目标：

1. 在 Linux 机器上，通过USB线连接到本设备，将创建一个新的fb设备

2. 在 Windows 机器上，通过USB线连接到本设备，将识别到一个新的显示器

上述两种方式都为当前的Host机器提供了主/拓展显示器支持。

3. 在不安装驱动的情况下，通过Python脚本来提供Host发送端支持。

下面这张图片是在 Raspberry Pi 上运行的效果：

{{< figure src="images/udd_rpi.jpg" alt="" >}}

工程还在开发中，目前还不支持触摸上报，您可以到如下仓库链接查看最新开发进度：

仓库链接：[https://gitee.com/embeddedboys/pico_dm_qd3503728_udd](https://gitee.com/embeddedboys/pico_dm_qd3503728_udd)

有关编译、烧录及使用的说明，请先查看上述仓库的 README 文件。

国内用户

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_udd
```

国外用户：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_udd
```

### RP2350 LVGL全屏刷新示例

如果使用局部刷新的话，那么在发送像素数据前，需要给屏幕驱动发送额外的命令来设置绘制的窗口，
从而降低了一定的刷新效率，而且我们的 8080 PIO 程序只支持设置 `DB0 - DB15` 和 `WR` 引脚，如果
涉及到 `命令/数据` 的切换写入，则需要在数据写入 PIO 前调用 `gpio_put` 来切换 `RS` 引脚以
告知屏幕驱动IC当前写入的是命令还是数据。

拓展板的分辨率为 480x320，使用 RGB565 格式，一整屏幕的像素占用 (480 x 320 x 2) = 307200 字节。
RP2350 具有 512KB 大小的 SRAM，所以我们可以在 RP2350 上启用 LVGL 的全屏刷新功能，每当 LVGL Screen
对象上有内容更新时，LVGL都会请求一次全屏刷新。

```c
static lv_disp_drv_t disp_drv;
    ...
disp_drv.full_refresh = 1;
    ...
lv_disp_drv_register(&disp_drv);
```

所以，当上述全屏刷新的逻辑建立后，我们可以调整屏幕初始化的流程如下：

初始化PIO -> 发送初始化序列 -> 调整显示方向 -> 设置刷新区域为全屏 -> 设置 RS 引脚为 1（数据）

这是上述流程的代码：

```c
static int ili9488_hw_init(struct ili9488_priv *priv)
{
    printf("initializing hardware...\n");

#if DISP_OVER_PIO
    i80_pio_init(priv->gpio.db[0], ARRAY_SIZE(priv->gpio.db), priv->gpio.wr);
#endif
    ili9488_gpio_init(priv);

    /* 发送初始化序列 */
    priv->tftops->init_display(priv);

    /* 调整显示方向 */
    priv->tftops->set_dir(priv, priv->display->rotate);

    /* clear screen to black */
    // priv->tftops->clear(priv, 0x0);

    /* 设置刷新区域为全屏 */
    priv->tftops->set_addr_win(priv, 0, 0, ILI9488_X_RES - 1, ILI9488_Y_RES - 1);

    /* 设置 RS 引脚为 1（数据） */
    gpio_put(priv->gpio.rs, 1);

    return 0;
}
```

然后我们的 LVGL 刷新回调函数就可以改为如下代码：

```c
#define write_buf(p, b, l) i80_write_buf(b, l)

void ili9488_flush(lv_disp_drv_t * disp_drv, const lv_area_t * area, lv_color_t * color_p)
{
    write_buf(&g_priv, (void *)color_p, lv_area_get_size(area) * 2);

    lv_disp_flush_ready(disp_drv);
}
```

每当该回调函数被调用时，都会处理一个长度为 `153600` 的 `RGB565` 像素buffer,
这是一个全屏 buffer， 所以不需要设置绘制区域，全程不涉及 RS 引脚切换， 完全
由 DMA + PIO 处理，这节省了一部分 CPU 资源。

拉取代码仓库：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_rp2350_lvgl_full_refresh.git
```

gitee 镜像待添加

### EEZ Studio LVGL 示例工程

Desktop / Embedded GUI development & Automation
桌面/嵌入式 GUI 开发和自动化

官方介绍：[https://www.envox.eu/studio/studio-introduction/](https://www.envox.eu/studio/studio-introduction/)

#### 家庭智能中控

{{< figure src="images/qd3503728_eez_studio.jpg" alt="" >}}

仓库链接：[https://gitee.com/embeddedboys/pico_dm_qd3503728_noos_eez_studio_demo](https://gitee.com/embeddedboys/pico_dm_qd3503728_noos_eez_studio_demo)

国内用户：

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_noos_eez_studio_demo
```

国外用户：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_noos_eez_studio_demo
```

#### 电源分析仪

{{< figure src="images/poweranalyzer_compressed.jpg" alt="" >}}

国内用户：

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_poweranylzer
```

国外用户：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_poweranylzer
```

### 8080屏模板工程

在开发本项目的过程中，其实我们还同时开发着其他类似项目，为了加快后续适配工作进度，我们开发了本工程，通过简单的配置就可以在多个lcd或触摸之间切换，这意味着您完全可以使用本工程在您自己的平台上开发。 😎 有关本工程的详细内容参见其readme文件。

仓库链接：[https://gitee.com/embeddedboys/pico_dm_8080_template](https://gitee.com/embeddedboys/pico_dm_8080_template)

国内用户：

```shell
git clone https://gitee.com/embeddedboys/pico_dm_8080_template
```

国外用户：

```shell
git clone https://github.com/embeddedboys/pico_dm_8080_template
```

#### 驱动支持情况

如果你需要添加新的屏幕或触摸，请参考这片文章 [点我跳转](../../porting/8080模板工程/)

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

##### 触摸驱动

- [x] FT6236U
- [x] NS2009
- [x] TSC2007
- [x] GT911

---

<!-- ### 如果上述版本都无法下载，尝试访问如下链接直接下载压缩包
链接：[https://pan.baidu.com/s/1m4WmPoHAZYiK3XwwXGrNDw?pwd=34mn](https://pan.baidu.com/s/1m4WmPoHAZYiK3XwwXGrNDw?pwd=34mn)

提取码：34mn


{{< callout context="note" title="说明" icon="info-circle" >}}
该方式的源码版本可能比较落后，最新版本以github仓库为准。
我们也会及时更新镜像链接版本。
{{< /callout >}} -->

### FreeRTOS

与裸机版本不同的是，我们又在其基础上添加了对 FreeRTOS 的支持，同时该工程支持 SMP，可同时使用RP2040的两个核心处理任务。 如果您惯用 FreeRTOS 开发，可以选择本工程。

仓库链接：[https://gitee.com/embeddedboys/pico_dm_qd3503728_freertos.git](https://gitee.com/embeddedboys/pico_dm_qd3503728_freertos.git)

国内用户

```shell
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_freertos.git
```

```shell
git clone https://github.com/embeddedboys/pico_dm_qd3503728_freertos.git
```

### ESP32-S3

目前计划支持引脚与树莓派Pico兼容的如下开发板：

- [x] [无名科技 ESP32-S3 Pico](https://www.nologo.tech/product/esp32/esp32s3/esp32s3Pico/esp32S3Pico.html)
- [ ] [WalnutPi 核桃派PicoW ESP32-S3](https://walnutpi.com/docs/walnutpi_picow/)
- [x] [Unknown ESP32-S3 Dev Board A](https://item.taobao.com/item.htm?_u=21m6r7hse5f8&id=749667421699)

[文档跳转链接](../../porting/esp32-s3/#说明)

### Linux

Linux 显示与触摸驱动，目前计划支持引脚与树莓派Pico兼容的如下开发板：

- [x] `Luckfox Pico`
- [x] `Luckfox Pico Max`
- [x] `Luckfox Lyra`
- [x] `Luckfox Lyra Plus`
- [x] `Milk-V Duo`
- [ ] `Milk-V Duo 256M`

[文档跳转链接](../../porting/linux/#说明)

### Micropython (Python)

仓库链接：[https://github.com/embeddedboys/lv_micropython.git](https://github.com/embeddedboys/lv_micropython.git)

我们针对LVGL的V8.3和V9两个版本都做了适配，用户可根据需求自行选择合适的版本。

因为micropython工程涉及太多子模块，所以不方便迁移到gitee，除非在网络环境允许的情况下或您对micropython有源码修改需求，否则不建议用户自行编译。

您可以到[Github Release](https://github.com/embeddedboys/lv_micropython/releases)界面直接下载我们编译好的固件直接烧录使用，位于对应Release的Assets菜单下，在对应的Release中也有介绍使用发法。 烧录方法可以参考[固件烧录](../../get-started/固件烧录/)

对于无法访问Github的用户，可以访问如下链接下载固件：

[http://embeddedboys.com/uploads/qd3503728/micropython/](http://embeddedboys.com/uploads/qd3503728/micropython/)

为了帮助用户更快上手micropython开发，我们还录制了一个如何搭建micropython开发环境的视频，可通过访问如下链接查看：

（待添加）

#### 测试方法

先下载你需要的固件，如果您需要LVGL V8.3版本，则下载如下版本：

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

{{< callout context="note" title="说明" icon="info-circle" >}}
上面的py测试文件是基于`LVGL V8.3`版本的，V9版本的测试文件请到[https://github.com/IotaHydrae/pico-py-lab/tree/main/v9](https://github.com/IotaHydrae/pico-py-lab/tree/main/v9)下查看
{{< /callout >}}

您也可以使用在线模拟器来调试micropython程序

V8.3: [https://sim.lvgl.io/v8.3/micropython/ports/javascript/index.html](https://sim.lvgl.io/v8.3/micropython/ports/javascript/index.html)

V9.0: [https://sim.lvgl.io/v9.0/micropython/ports/webassembly/index.html](https://sim.lvgl.io/v9.0/micropython/ports/webassembly/index.html)

可在[lv_mpy_examples_v8](https://github.com/uraich/lv_mpy_examples_v8)这个仓库查看v8.3版本的lvgl micropython例子，因为v9版本较新，暂时没有例程参考。

### Arduino

#### 所需硬件

- Raspberry Pi Pico (with BOOTSEL button)
- 一根 Type-C USB 或 Micro-USB 线缆

底层驱动支持情况：

- [x] Display via PIO + DMA
- [x] Touch

#### 在你开始之前

0.  通过git或者下载zip来获取本工程

    ```bash
    git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_arduino
    ```

1.  在 Arduino IDE 中安装 pico 开发板

    > 参考自 [https://github.com/earlephilhower/arduino-pico](https://github.com/earlephilhower/arduino-pico)

    打开 Arduino IDE 并转到 `文件->首选项`。

    在弹出的对话框中，在 `其他开发板管理器地址` 字段中输入以下 URL：

    ```bash
    https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json
    ```

    {{< figure src="images/board_url.png" alt="" >}}

    单击 `确定` 关闭对话框。

    在 IDE 中转到 `工具->开发板->开发板管理器`

    在搜索框中输入`pico`，并选择`安装`：

    {{< figure src="images/install.png" alt="" >}}

    等待安装完成

2.  通过 Arduino IDE 安装 lvgl 和 TFT_eSPI 库
    - TFT_eSPI
    - lvgl (version == 8.4.0)

3.  将 `TFT_eSPI/User_Setup.h` 替换成本工程中提供的

    ```bash
    cd pico_dm_qd3503728_arduino
    cp User_Setup.h ~/Arduino/libraries/TFT_eSPI/
    ```

4.  将 `lv_conf.h` 拷贝至 `Arduino/libraries` 目录下

    ```bash
    cd pico_dm_qd3503728_arduino
    cp lv_conf.h ~/Arduino/libraries/
    ```

5.  如果你想要构建 lvgl 的 demos, 将 `lvgl/demos`
    目录拷贝至 `lvgl/src` 目录下， examples 也一样

        ```bash
        cd ~/Arduino/libraries/lvgl
        cp demos/ -r src/
        cp examples -r src/
        ```

这时 `Arduino` 目录看起来是这样的：

```bash
libraries\
    lvgl\
    TFT_eSPI\
        User_Setup.h
    lv_conf.h
```

> (`Arduino`目录在 Windows 上通常默认位于 `C\Users\your_username\Documents\Arduino` , 在 linux 上通常位于`~/Arduino`)

6. 在`Arduino IDE`中, 找到 `File->Open` 并且打开本工程中的 `main/main.ino` 文件

7. 上传工程到 Pico

   当你第一次上传工程时，你需要按下 Pico 的 BOOTSEL 按钮，然后插入你的电脑。此外，在你修改了工程后，你可以直接上传到你的 Pico 上。

   每次上传工程时，建议选择正确的 COM 端口。

### embedded_graphics (Rust)

[https://github.com/embedded-graphics/embedded-graphics](https://github.com/embedded-graphics/embedded-graphics)

`embedded_graphics` 是一款专注于内存受限的嵌入式设备的二维图形库，基于Rust开发。 我们将从该工程基础上移植[Slint](#slint-rust)。

底层驱动支持情况：

- [x] Display via GPIO
- [x] Display via PIO
- [ ] Display via PIO + DMA
- [x] Touch

仓库链接：[https://github.com/embeddedboys/pico_dm_qd3503728_embedded_graphics](https://github.com/embeddedboys/pico_dm_qd3503728_embedded_graphics)

国内用户

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_embedded_graphics
```

国外用户

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_embedded_graphics
```

编译烧录参考[编译及配置](../编译及配置/#embedded_graphics)

#### lv_binding_rust

我们又在此基础上移植了[lv_binding_rust](https://github.com/lvgl/lv_binding_rust/)的示例，但是该工程目前相当混乱，而且速度也不是很可观，在处理好相关问题之后我们会提供示例。 我们不建议将该工程用于实际项目中，因为相关负责人已经表示不再积极维护该项目。

### Slint (Rust)

[https://slint.dev/](https://slint.dev/)

我们添加了一个初步支持的slint移植，可查看如下仓库

[https://github.com/embeddedboys/pico_dm_qd3503728_slint_mcu](https://github.com/embeddedboys/pico_dm_qd3503728_slint_mcu)

这部分的文档还在整理中，可先查看如上仓库的 [README](https://github.com/embeddedboys/pico_dm_qd3503728_slint_mcu/blob/main/README.md)

底层驱动支持情况：

- [x] Display via GPIO
- [x] Display via PIO
- [ ] Display via PIO + DMA
- [x] Touch

国内用户

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_slint_mcu
```

国外用户

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_slint_mcu
```

点击查看[编译及配置](../编译及配置/#slint)

### Zephyr

[关于Zephyr](https://docs.zephyrproject.org/latest/introduction/index.html)

Zephyr RTOS 是基于一个小型内核设计的，用于资源有限的嵌入式系统: 从简单的嵌入式环境传感器和 LED 可穿戴设备到复杂的嵌入式控制器、智能手表和物联网无线应用。

最大的特点就是用linux开发的方式来开发MCU。

[https://docs.zephyrproject.org/latest/boards/raspberrypi/rpi_pico/doc/index.html](https://docs.zephyrproject.org/latest/boards/raspberrypi/rpi_pico/doc/index.html)

工程开发完成，整理中。。。

国内用户：

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_zephyr
```

国外用户：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_zephyr
```

### Nuttx

[关于Nuttx](https://nuttx.apache.org/docs/latest/introduction/about.html)

[https://nuttx.apache.org/docs/latest/platforms/arm/rp2040/index.html](https://nuttx.apache.org/docs/latest/platforms/arm/rp2040/index.html)

已提上日程，调研中。。。

### hagl

HAGL 是一个轻量级的硬件无关图形库。它支持基本几何图元、位图、位图传送、固定宽度字体。
该库试图保持轻量级，但针对功能相当强大的微芯片，例如 ESP32。没有动态分配。

[hagl: https://github.com/tuupola/hagl](https://github.com/tuupola/hagl)

相关链接：
[https://github.com/tuupola/hagl_pico_mipi](https://github.com/tuupola/hagl_pico_mipi)

工程开发完成，资料整理中，请先参考各仓库中的 `README.md` 文件

#### effects 场景渲染测试

{{< figure src="images/hagl_effects_0.png" alt="" >}}
{{< figure src="images/hagl_effects_1.png" alt="" >}}

国内用户：

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_hagl_effects
```

国外用户：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_hagl_effects
```

#### GFX 性能测试示例

{{< figure src="images/hagl_gfx.jpg" alt="" >}}

国内用户：

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_hagl_gfx
```

国外用户：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_hagl_gfx
```

### AWTK

AWTK 是 Toolkit AnyWhere 的缩写，是 ZLG 开发的一个开源 GUI 引擎。
它是一个用于嵌入式系统、 WEB、迷你程序、移动电话和 PC 的跨平台 GUI 引擎。
它是一个功能强大、高效、可靠和易于使用的 GUI 引擎，为用户设计美观的 GUI 应用程序。

{{< figure src="images/awtk.png" alt="" >}}

[awtk: https://github.com/zlgopen/awtk](https://github.com/zlgopen/awtk)

[关于 AWTK](https://awtk.zlg.cn/docs/awtk_docs/FAQ/1.AWTK.html)

相关链接：
[https://github.com/zlgopen/awtk-pico](https://github.com/zlgopen/awtk-pico)
[https://github.com/zlgopen/awtk-previewer](https://github.com/zlgopen/awtk-previewer)
[AWTK移植及裁剪指南.pdf](https://awtk.zlg.cn/docs/awtk_docs/_public_/AWTK%E7%A7%BB%E6%A4%8D%E5%8F%8A%E8%A3%81%E5%89%AA%E6%8C%87%E5%8D%97.pdf)

工程开发完成，资料整理中，请先参考仓库中的 `README.md` 文件

国内用户：

```bash
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_awtk
```

国外用户：

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_awtk
```

### uMac

Micro Mac, minimalist Macintosh 128K emulator

{{< figure src="images/pico_dm_qd3503728_umac.JPG" alt="" >}}

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_umac.git
```

工程中，请先参考仓库中的 [`README.md`](https://github.com/embeddedboys/pico_dm_qd3503728_umac/blob/main/README.md) 文件

### SGL

SGL (Small Graphics Library) 是一个轻量级且快速的图形库，专为MCU级别处理器提供美观轻量的GUI（图形用户界面）。 请参考 docs 目录获取文档。

官方仓库链接：[https://github.com/sgl-org/sgl](https://github.com/sgl-org/sgl)

##### SGL UI库特点

- 轻量级，最小只需要3KBytes RAM和15KBytes ROM即可运行
- 部分帧缓冲支持，最小只需要一行屏幕分辨率的缓冲即可
- 包围盒+贪心算法的脏矩形算法
- 支持帧缓冲控制器，直接向帧缓冲控制器写入数据，零拷贝
- 颜色深度支持: 8bit, 16bit, 24bit, 32bit
- 现代化的字体取模工具
- SGL自己的UI设计器，图形化拖动绘制界面后一键可生成代码

##### 最低硬件要求

| Flash 大小 | RAM 大小 |
| ---------- | -------- |
| 15KB       | 3KB      |

##### 示例工程

工程链接：[https://github.com/embeddedboys/pico_dm_qd3503728_sgl](https://github.com/embeddedboys/pico_dm_qd3503728_sgl)

有关此移植的更多信息请参考工程 README 文件

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_sgl.git
cd pico_dm_qd3503728_sgl

git submodule update --init

mkdir -p build && cd build
cmake -DPICO_BOARD=pico2 .. -G Ninja
ninja
```

### uGUI

µGUI 是一个免费且开源的嵌入式系统图形库。它是平台无关的（platform-independent），因此可以很容易地移植到几乎任何微控制器系统上。
只要显示设备能够显示图形，µGUI 就不受特定显示技术的限制。因此，它支持多种显示技术，例如 LCD、TFT、电子纸（E-Paper）、LED 和 OLED。
整个模块由三个文件组成：`ugui.c` `ugui.h` `ugui_config.h`

官方仓库链接：[https://github.com/achimdoebler/UGUI](https://github.com/achimdoebler/UGUI)

##### 示例工程

工程链接：[https://github.com/embeddedboys/pico_dm_qd3503728_ugui](https://github.com/embeddedboys/pico_dm_qd3503728_ugui)
