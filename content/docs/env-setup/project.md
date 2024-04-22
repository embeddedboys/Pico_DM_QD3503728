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

## 基于PICO-SDK的

### 裸机版本

该版本完全基于官方 Pico C-SDK 开发，仅添加了LVGL的支持，所以如果您想要在本项目基础上进行原生二次开发，可以选择该裸机工程。

国内用户
```shell
git clone https://gitee.com/embeddedboys/pico_dm_qd3503728_noos
```

```shell
git clone https://github.com/embeddedboys/pico_dm_qd3503728_noos
```

### FreeRTOS 版本

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
- [x] ILI9488
- [ ] ILI9486
- [ ] ILI9341
- [x] R61581
- [x] 1P5623
- [ ] LG4572B
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

## 其他版本

 😋 我们正在开发中，包括但不限于如下工程：

- [x] Micropython
- [ ] Arduino
- [ ] Nuttx

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

1. 烧录固件到pico核心板

```
使用`firmware.uf2`或其他格式固件，按照适合您的方式烧录
```

2. 使用Thonny或MicroPico上传库文件

```
可以使用Thonny或者MicroPico的内置功能，保存`lv_utils.py`到核心板文件系统中。
```

3. 运行测试

```
执行`ili9488_test.py`，然后此时屏幕应有“Hello World”按钮出现。
```

### Arduino
（开发中）开发进度：
- [x] 研究使用platformio Arduino开发rp2040程序
- [ ] 搭建工程