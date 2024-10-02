---
title: "TFT模块"
description: ""
summary: ""
date: 2024-03-23T00:11:05Z
lastmod: 2024-03-23T00:11:05Z
draft: false
weight: 701
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

### FPC-ZH096G1321

{{< picture
  src="images/bonus-dm-0.jpg"
  class="rounded-3"
  alt=""
>}}

| | |
| --- | --- | --- |
| 驱动 | ST7735S | |
| 分辨率 | 160x80 | |
| 引脚定义 |  | |
| 0 | GND | 电源地 |
| 1 | VCC | 电源输入 |
| 2 | SCL(SPI_SCK) | SPI时钟 |
| 3 | SDA(SPI_MOSI) | SPI MOSI |
| 4 | RES | 复位 |
| 5 | DC | 数据/命令选择 |
| 6 | CS | SPI 片选 |
| 7 | BLK | 背光控制 |

参考代码：[st7735s-lvgl](https://github.com/IotaHydrae/rpi-pico-lab/tree/main/st7735s-lvgl)

### SINQ13001BL-A1

{{< figure
  src="images/bonus-dm-1.jpg"
  class="rounded-3"
  alt=""
>}}

| | |
| --- | --- |
| 驱动 | ST7789V |
| 分辨率 | 240x240 |

| | |
| --- | --- | --- |
| 驱动 | ST7735S | |
| 分辨率 | 160x80 | |
| 引脚定义 |  | |
| 0 | GND | 电源地 |
| 1 | VCC | 电源输入 |
| 2 | SCL(SPI_SCK) | SPI时钟 |
| 3 | SDA(SPI_MOSI) | SPI MOSI |
| 4 | RES | 复位 |
| 5 | DC | 数据/命令选择 |
| 6 | CS | SPI 片选 |
| 7 | BLK | 背光控制 |

参考代码：[st7789v-lvgl](https://github.com/IotaHydrae/rpi-pico-lab/tree/main/st7789v-lvgl)