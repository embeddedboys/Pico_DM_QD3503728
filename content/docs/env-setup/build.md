---
title: "编译及配置"
description: ""
summary: ""
date: 2024-03-09T07:30:43Z
lastmod: 2024-03-09T07:30:43Z
draft: false
weight: 205
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

如果您是从github，或者gitee拉取的工程，在进行编译之前，应该先更新子模块，方式如下：
```shell
# 位于工程根目录下
git submodule update --init
```
此方式对网络环境有要求，子模块源来自github，可以通过代理等方式。
我们正在尝试将子模块迁移至gitee，保证国内用户可以正常下载工程。

如果不满足上述条件，可以直接下载对应工程压缩包（[位于上一章节](/docs/env-setup/选择工程)），我们会不定期进行更新。

## 编译工程

### pico_dm_qd3503728_noos

裸机工程

#### 目录结构

```shell
backlight.c   # 背光驱动
CMakeLists.txt  # 工程cmake配置文件
factory   # 工厂测试程序
ft6236.c  # 触摸驱动
i2c_tools.c # i2c工具
ili9488.c   # 显示驱动
include   # 头文件
LICENSE   # 许可证
lv_conf.h   # lvgl配置头文件
lvgl      # lvgl源码
main.c    # 程序入口
pico_sdk_import.cmake   # pico-sdk前置文件
pio       # pio相关驱动
porting   # lvgl移植文件
```

#### 编译生成固件
```
cd pico_dm_qd3503728_noos

mkdir -p build
cd build
cmake ..
make -j12
```

### pico_dm_qd3503728_freertos

只针对于本产品的freertos工程

#### 根目录结构
```shell
CMakeLists.txt  # 根目录cmake配置
LICENSE         # 许可证
README.md       # 自述文件
include/        # 头文件
lib/            # 第三方库
pico_sdk_import.cmake # pico sdk前置文件
src/            # 工程源码
```

#### src目录结构
```shell
CMakeLists.txt    # 工程主要cmake配置
FreeRTOSConfig.h  # FreeRTOS 配置文件
backlight.c       # 背光驱动
factory/          # 工厂测试相关
ft6236.c          # 触摸驱动
i2c_tools.c       # I2C 工具
ili9488.c         # 显示驱动
lv_conf.h         # lvgl 配置文件
lvgl/             # lvgl 源码
main.c            # 程序入口
pio/              # PIO 相关驱动
porting/          # lvgl 移植文件
```

#### 编译生成固件
```shell
cd pico_dm_qd3503728_freertos

mkdir -p build
cd build
cmake ..
make -j12
```

### pico_dm_8080_template

针对多个产品的工程模板

#### 根目录结构
```shell
CMakeLists.txt  # 根目录cmake配置
LICENSE         # 许可证
README.md       # 自述文件
include/        # 头文件
lib/            # 第三方库
pico_sdk_import.cmake # pico sdk前置文件
src/            # 工程源码
```

#### src目录结构
```shell
backlight.c
cmake
CMakeLists.txt
factory
FreeRTOSConfig.h
ft6236.c
gt911.c
i2c_tools.c
indev.c
lv_conf.h
lvgl
main.c
ns2009.c
pio
porting
tft_1p5623.c  # 1p5623 面板驱动
tft.c
tft_ili9488.c # ili9488 显示驱动
tft_r61581.c  # r61581 显示驱动
tft_st6201.c  # st6201 显示驱动
tft_st7789.c # st7789 显示驱动
tsc2007.c   # tsc2007 电阻屏触摸驱动
```

## 烧录

可参考[固件烧录](/docs/get-started/固件烧录/)章节，此处给出示例

### UF2

按住核心板上的BOOTSEL键，然后插入USB线缆，或者在插入线缆的情况下按下RUN按键，
此时RP2040将进入BOOTROM USB下载模式。

```shell
cp src/rp2040-freertos-template.uf2 /media/$USER/RPI-RP2/
```

{{< callout context="note" title="说明" icon="info-circle" >}}
Windows 用户可右键 uf2 文件选择发送到 RPI-RP2
{{< /callout >}}


### openocd

```shell
sudo openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000" -c "program rp2040-freertos-template.elf verify reset exit
```

{{< callout context="note" title="说明" icon="info-circle" >}}
WSL用户需要先将daplink连接至WSL中，可使用[usbipd](https://github.com/dorssel/usbipd-win)
{{< /callout >}}

## CMakeLists.txt 配置说明

此处列出可供用户自定义修改的选项

### 是否开启超频
```shell
set(OVERCLOCK_ENABLED 0)    # 1: enable, 0: disable
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

您也可根据核心板稳定程度，自行调整超频幅度。

{{< callout context="caution" title="注意" icon="alert-triangle" >}}
在设置CPU频率时，还应注意是否符合Flash的工作频率，公式如下：

Flash Clock = SYS_CLK / PICO_FLASH_SPI_CLKDIV

PICO_FLASH_SPI_CLKDIV的默认值为2

简单来说，当CPU频率处于266MHz的情况下，Flash工作频率为133MHz

在本工程中，当CPU频率大于266MHz时，PICO_FLASH_SPI_CLKDIV自动修改为4，否则将超出Flash最大工作频率

{{< /callout >}}

### LCD 相关

可在此处配置引脚、时钟、是否启用PIO等。
若存在缺失配置项，则表示该工程不支持调整。
```shell
# LCD Pins for 8080 interface
set(LCD_PIN_DB_BASE  0)  # 8080 LCD 数据总线第0脚
set(LCD_PIN_DB_COUNT 16) # 8080 LCD 数据总线宽度
set(LCD_PIN_CS  18)  # 8080 LCD 片选引脚
set(LCD_PIN_WR  19)  # 8080 LCD 写信号引脚
set(LCD_PIN_RS  20)  # 8080 LCD 数据/寄存器选择引脚
set(LCD_PIN_RST 22)  # 8080 LCD 复位引脚
set(LCD_PIN_BL  28)  # 8080 LCD 背光引脚
set(DISP_OVER_PIO 1) # LCD驱动模式 1: PIO, 0: GPIO
set(PIO_USE_DMA   1)   # 是否启用DMA 1: use DMA, 0: not use DMA
set(I80_BUS_WR_CLK_KHZ 50000) # 8080 LCD 写信号频率
```