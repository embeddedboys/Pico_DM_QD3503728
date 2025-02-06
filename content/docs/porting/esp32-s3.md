---
title: "ESP32-S3"
description: ""
summary: ""
date: 2024-03-27T22:46:10Z
lastmod: 2024-03-27T22:46:10Z
draft: false
weight: 1103
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 说明

本文用于介绍如何将此拓展板部署到 ESP32-S3 平台中，您可根据开发习惯选择 Arduino 或 ESP-IDF 开发。

- [x] [无名科技 ESP32-S3 Pico](https://www.nologo.tech/product/esp32/esp32s3/esp32s3Pico/esp32S3Pico.html)
- [ ] [WalnutPi 核桃派PicoW ESP32-S3](https://walnutpi.com/docs/walnutpi_picow/)
- [x] [Unknown ESP32-S3 Dev Board A](https://item.taobao.com/item.htm?_u=21m6r7hse5f8&id=749667421699)

### Arduino

工程开发中，暂不支持。

### ESP-IDF

- Graphics & Touch Driver : [LovyanGFX](https://github.com/lovyan03/LovyanGFX)

- UI / Widgets : [LVGL 9.1.0](https://github.com/lvgl/lvgl)

- Framework : [ESP-IDF 5.2.3](https://github.com/espressif/esp-idf/)

#### 搭建工程

##### 1. 拉取工程并更新子模块

```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_esp32s3_idf
cd pico_dm_qd3503728_esp32s3_idf
git submodule update --init
```

##### 2. 准备 ESP-IDF 环境

Tested with ESP-IDF v5.2.3. Other versions may work as well.

```bash
mkdir -p ~/esp && cd ~/esp
git clone https://github.com/espressif/esp-idf.git -b release/v5.2
cd esp-idf
./install.sh
```

Make sure you have esp-idf exported, e.g.:
```bash
source ~/esp/esp-idf/export.sh
idf.py set-target esp32s3
```

##### 3. 配置并编译烧录

这个项目的设计是兼容多个核心板配置，您需要打开 `main/LGFX_MakerFabs_Parallel_S3.hpp`。 并修改 `DEFAULT_CORE_BOARD_MODEL` 以适应您的核心板。以下是一些可供参考的选择：

- Nologo tech ESP32-S3 Pico
- Walnutpi PicoW
- Unknown ESP32 S3 Dev Board A

```c
#define NOLOGO_ESP32S3_PICO   1
#define WALNUTPI_PICOW        2
#define UNKNOWN_ESP32S3_PICO  3

#ifndef DEFAULT_CORE_BOARD_MODEL
  #define DEFAULT_CORE_BOARD_MODEL NOLOGO_ESP32S3_PICO
#endif
```

编译烧录：
```bash
idf.py build flash monitor
```

#### 引脚定义

##### [Nologo tech ESP32-S3 Pico](https://www.nologo.tech/product/esp32/esp32s3/esp32s3Pico/esp32S3Pico.html)

{{< figure src="images/esp32s3picofoot_compressed.png" alt="" >}}

##### [Walnutpi PicoW](https://walnutpi.com/docs/walnutpi_picow/)

{{< figure src="images/walnutpi_picowfoot_compressed.png" alt="" >}}

##### [Unknown ESP32 S3 Dev Board A](https://item.taobao.com/item.htm?_u=21m6r7hse5f8&id=749667421699)

```c
#define TFT_PIN_BLK   9
#define TFT_PIN_WR    3
#define TFT_PIN_RD    41
#define TFT_PIN_RS    4
#define TFT_PIN_RST   6

#define TFT_PIN_D0    43
#define TFT_PIN_D1    44
#define TFT_PIN_D2    38
#define TFT_PIN_D3    39
#define TFT_PIN_D4    40
#define TFT_PIN_D5    41
#define TFT_PIN_D6    42
#define TFT_PIN_D7    21
#define TFT_PIN_D8    20
#define TFT_PIN_D9    19
#define TFT_PIN_D10   18
#define TFT_PIN_D11   17
#define TFT_PIN_D12   14
#define TFT_PIN_D13   13
#define TFT_PIN_D14   12
#define TFT_PIN_D15   11

#define TP_PIN_SDA    7
#define TP_PIN_SCL    8
#define TP_PIN_INT    5
```

#### 版权声明

此工程基于 [Makefabs]() 的 [makerfabs-parallel-tft-lvgl-lgfx](https://github.com/radiosound-com/makerfabs-parallel-tft-lvgl-lgfx) 修改而来。