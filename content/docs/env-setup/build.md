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

目录结构

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

编译生成固件
```
cd pico_dm_qd3503728_noos

mkdir -p build
cd build
cmake ..
make -j12
```

### pico_dm_qd3503728_freertos

只针对于本产品的freertos工程

根目录结构
```shell
CMakeLists.txt  # 根目录cmake配置
LICENSE         # 许可证
README.md       # 自述文件
include/        # 头文件
lib/            # 第三方库
pico_sdk_import.cmake # pico sdk前置文件
src/            # 工程源码
```

src目录结构
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

编译生成固件
```shell
cd pico_dm_qd3503728_freertos

mkdir -p build
cd build
cmake ..
make -j12
```

### pico_dm_8080_template

针对多个产品的工程模板

根目录结构
```shell
CMakeLists.txt  # 根目录cmake配置
LICENSE         # 许可证
README.md       # 自述文件
include/        # 头文件
lib/            # 第三方库
pico_sdk_import.cmake # pico sdk前置文件
src/            # 工程源码
```

src目录结构
```shell
backlight.c     # 背光驱动
cmake           # 一些cmake配置文件，板级或驱动级配置
CMakeLists.txt  # src cmake配置
factory       # 工厂测试相关
FreeRTOSConfig.h  # FreeRTOS配置文件
ft6236.c    # ft6236 电容触摸屏驱动
gt911.c     # gt911 电容触摸屏驱动
i2c_tools.c # i2c工具
indev.c     # 输入驱动框架
lv_conf.h   # lvgl配置头文件
lvgl        # lvgl源码
main.c      # 程序入口
ns2009.c    # ns2009 电阻触摸屏驱动
pio         # pio相关驱动
porting     # lvgl移植文件
tft_1p5623.c  # 1p5623 面板驱动
tft.c         # 显示驱动框架
tft_ili9488.c # ili9488 显示驱动
tft_r61581.c  # r61581 显示驱动
tft_st6201.c  # st6201 显示驱动
tft_st7789.c # st7789 显示驱动
tsc2007.c   # tsc2007 电阻触摸屏驱动
```

编译生成固件

```shell
cd pico_dm_8080_template

mkdir -p build
cd build
cmake ..
make -j12
```

### MicroPython

拉取并进入工程目录
```bash
git clone https://github.com/embeddedboys/lv_micropython.git
cd lv_micropython
```

切换你需要编译的分支版本，对于LVGL V8.3
```bash
git checkout release/v8
```
对于LVGL V9
```bash
git checkout release/v9
```
{{< callout context="note" title="说明" icon="info-circle" >}}
如果已经进行过编译，若切换分支，则需要在切换分支之后运行如下命令同步切换子模块版本
```bash
git submodule update
```
{{< /callout >}}


若拉取模块过程失败则需重新执行，如果还是报错，请在对应make命令最后加上clean进行清理，然后重新执行对应命令。

{{< tabs "install-sdk-requirements" >}}
{{< tab "Ubuntu" >}}

```bash {title="通用步骤"}
sudo apt install cmake build-essenital gcc-arm-none-eabi -y

git submodule update --init --recursive lib/lv_bindings
make -C ports/rp2 BOARD=PICO submodules
make -j -C mpy-cross
make -j -C ports/rp2 BOARD=PICO USER_C_MODULES=../../lib/lv_bindings/bindings.cmake
```

{{< /tab >}}

{{< tab "Fedora" >}}

```bash {title="通用步骤"}
sudo dnf install make gcc gcc-g++ cmake arm-none-eabi-gcc-cs arm-none-eabi-gcc-cs-c++ arm-none-eabi-newlib -y

git submodule update --init --recursive lib/lv_bindings
make -C ports/rp2 BOARD=PICO submodules
make -j -C mpy-cross
make -j -C ports/rp2 BOARD=PICO USER_C_MODULES=../../lib/lv_bindings/bindings.cmake
```

{{< /tab >}}
{{< /tabs >}}

### embedded_graphics

#### 环境配置
1. 先安装Rust环境，在linux或者wsl中执行如下命令
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

2. 安装需要的工具链
```bash
rustup self update
rustup update stable
rustup target add thumbv6m-none-eabi
```

3. 安装调试烧录工具
```bash
# Useful to creating UF2 images for the RP2040 USB Bootloader
cargo install elf2uf2-rs --locked
# Useful for flashing over the SWD pins using a supported JTAG probe
cargo install --locked probe-rs-tools
```

#### 编译及烧录

在安装完上一节中的软件包之后，执行如下命令编译工程
```bash
cargo build -r
```

> -r 代表 release 编译

编译指定example
```bash
cargo build -r --example demo-text-tga
```

有两种方式将编译好的文件烧录到Pico

1. 通过CMSIS-DAP调试器

你可能需要先配置udev rules才能让cmsis-dap得以识别到，复制工程目录下的`50-cmsis-dap.rules`，
到`/etc/udev/rules.d/`路径下，然后执行
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

使用如下命令通过调试器烧录，并监视RTT调试信息
```bash
probe-rs run --chip RP2040 --protocol swd target/thumbv6m-none-eabi/release/rp2040-project-template
```

2. 通过RP2040的bootloader UF2烧录

按住核心板的`BOOTSEL`按键，插入USB线，或者在连接有线的情况下，按下拓展板上的复位键，让RP2040进入UF2下载模式，再通过如下命令将UF2文件下载至RP2040。
```bash
elf2uf2-rs -d target/thumbv6m-none-eabi/release/rp2040-project-template
```

或者你可以简单的运行如下命令，编译并将文件烧录到RP2040。

1. 修改`.cargo/config.toml`中的`runner`配置为符合您当前情况的配置。
```toml
runner = "probe-rs run --chip RP2040 --protocol swd"
# runner = "elf2uf2-rs -d"
```

2. 运行target
```bash
cargo run -r
```

运行指定example
```bash
cargo run -r --example demo-text-tga
```


### Slint

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
openocd -f interface/cmsis-dap.cfg -c "adapter speed 5000" -f target/rp2040.cfg -s tcl -c "program src/rp2040-freertos-template.elf verify reset exit"
```
program后面跟的参数是需要烧录的elf文件，当然也可以烧录bin文件，方式如下
```shell
openocd -f interface/cmsis-dap.cfg -c "adapter speed 10000" -f target/rp2040.cfg -s tcl -c "program src/rp2040-freertos-template.bin verify reset exit 0x10000000"
```
写入到`0x10000000`处地址，也就是rp2040映射Flash的地方

如何制作一个picoprobe（debugprobe）调试器，可以参考[这个章节](/docs/get-started/固件烧录/#debugprobe)

{{< callout context="note" title="说明" icon="info-circle" >}}
WSL用户需要先将daplink连接至WSL中，可使用[usbipd](https://github.com/dorssel/usbipd-win)
{{< /callout >}}

## CMakeLists.txt 配置说明

此处列出可供用户自定义修改的选项

### 是否开启超频
```cmake
set(OVERCLOCK_ENABLED 0)    # 1: enable, 0: disable
```
{{< callout context="caution" title="注意" icon="alert-triangle" >}}
过度超频可能会导致核心板稳定性下降，但并不会对拓展版造成影响。

适当的超频可以达到更流畅的运行效果，用户自行承担其风险。
{{< /callout >}}

### 超频预设
```cmake
# Overclocking profiles
#      SYS_CLK  | FLASH_CLK | Voltage
#  1  | 266MHz  |  133MHz   |  1.10(V) (default, stable, recommended for most devices)
#  2  | 240MHz  |  120MHZ   |  1.10(V) (more stable)
#  3  | 360MHz  |  90MHz    |  1.20(V)
#  4  | 400MHz  |  100MHz   |  1.30(V)
#  5  | 420MHz  |  105MHz   |  1.30(V)
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
```cmake
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

### 选择显示驱动
```cmake
# LCD driver type
set(LCD_DRV_USE_ST7789  0)
set(LCD_DRV_USE_ILI9488 0)
set(LCD_DRV_USE_ILI9806 1)
set(LCD_DRV_USE_R61581  0)
set(LCD_DRV_USE_ST6201  0)
set(LCD_DRV_USE_1P5623  0)
```

### 选择触摸驱动
```cmake
# Input device driver type
set(INDEV_DRV_USE_FT6236  0)
set(INDEV_DRV_USE_NS2009  0)
set(INDEV_DRV_USE_TSC2007 1)
set(INDEV_DRV_USE_GT911   0)
```

### 杂项
```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # 设置自动生成compile_commands.json
pico_enable_stdio_usb(${PROJECT_NAME} 0)  # 设置stdio通过usb cdc输出
pico_enable_stdio_uart(${PROJECT_NAME} 1) # 设置stdio通过uart输出
pico_add_extra_outputs(${PROJECT_NAME}) # 输出额外的编译文件，例如uf2等
```