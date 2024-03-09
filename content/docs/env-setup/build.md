---
title: "编译及配置"
description: ""
summary: ""
date: 2024-03-09T07:30:43Z
lastmod: 2024-03-09T07:30:43Z
draft: false
weight: 840
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 编译工程

此处以FreeRTOS版本工程为例

```shell
cd pico_dm_qd3503728_freertos

mkdir build
cd build
cmake ..
make -j12
```

## 烧录

可参考[固件烧录](/docs/get-started/固件烧录/)章节，此处给出示例

### UF2

```shell
cp src/rp2040-freertos-template.uf2 /media/$USER/RPI-RP2/
```

### openocd

```shell
sudo openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000" -c "program rp2040-freertos-template.elf verify reset exit
```

## cmake配置说明（通用）

此处列出可供用户自定义修改的选项

### 是否开启超频
```shell
set(OVERCLOCK_ENABLED 1)    # 1: enable, 0: disable
```
{{< callout context="caution" title="注意" icon="alert-triangle" >}}
过度超频可能会导致设备稳定性下降，用户自行承担其风险。
{{< /callout >}}

### 超频预设
```shell
# Overclocking profiles
#      SYS_CLK  | FLASH_CLK | Voltage
#  1  | 266MHz  |  133MHz   |  1.10(V) (default, stable, recommended for most devices)
#  2  | 240MHz  |  120MHZ   |  1.10(V) (more stable)
#  3  | 360MHz  |  90MHz    |  1.20(V)
#  4  | 400MHz  |  100MHz   |  1.25(V)
set(OVERCLOCK_PROFILE 4)
```

### LCD 相关

可在此处配置引脚、时钟、是否启用PIO等。
```shell
# LCD Pins for 8080 interface
set(LCD_PIN_DB_BASE  0)  # 8080 LCD data bus base pin
set(LCD_PIN_DB_COUNT 16) # 8080 LCD data bus pin count
set(LCD_PIN_CS  18)  # 8080 LCD chip select pin
set(LCD_PIN_WR  19)  # 8080 LCD write pin
set(LCD_PIN_RS  20)  # 8080 LCD register select pin
set(LCD_PIN_RST 22)  # 8080 LCD reset pin
set(LCD_PIN_BL  28)  # 8080 LCD backlight pin
set(DISP_OVER_PIO 1) # 1: PIO, 0: GPIO
```