---
title: "8080模板工程"
description: ""
summary: ""
date: 2024-04-20T05:40:21+08:00
lastmod: 2024-04-20T05:40:21+08:00
draft: false
weight: 1105
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

如果你希望将本工程移植到自己的电路板上，那么你可能会对本文章感兴趣。

本文将演示如何使用[pico_dm_8080_template](https://github.com/embeddedboys/pico_dm_8080_template)工程，适配一个全新的硬件组合，以`ILI9488` + `FT6236U`为例。

## 部署工程

## 显示屏

**1.新建一个名为tft_ili9488.c的文件**

**2.在文件中新建一个`tft_display`**

重写需要的`tftops`函数
```c
#include "tft.h"
#include "debug.h"

#if LCD_DRV_USE_ILI9488

...

static struct tft_display ili9488 = {
    .xres   = TFT_X_RES,
    .yres   = TFT_Y_RES,
    .bpp    = 16,
    .backlight = 100,
    .tftops = {
#if LCD_PIN_DB_COUNT == 8
        .write_reg = tft_write_reg8,
        .video_sync = tft_video_sync,
#else
        .write_reg = tft_write_reg16,
#endif
        .init_display = tft_ili9488_init_display,
    },
};

int tft_driver_init(void)
{
    tft_probe(&ili9488);
    return 0;
}

#endif
```

**3. 在src/cmake 目录下新建一个ili9488.cmake 文件**

在此处定义需要的变量，这些变量会覆盖`src/CMakeLists.txt`中的默认属性
```cmake
set(LCD_PIN_DB_BASE  0)  # 8080 LCD data bus base pin
set(LCD_PIN_DB_COUNT 16) # 8080 LCD data bus pin count
set(LCD_PIN_CS  29)  # 8080 LCD chip select pin
set(LCD_PIN_WR  19)  # 8080 LCD write pin
set(LCD_PIN_RS  20)  # 8080 LCD register select pin
set(LCD_PIN_RD  18)  # 8080 LCD read pin
set(LCD_PIN_RST 22)  # 8080 LCD reset pin
set(LCD_PIN_BL  28)  # 8080 LCD backlight pin
set(LCD_HOR_RES 480)
set(LCD_VER_RES 320)
set(DISP_OVER_PIO 1) # 1: PIO, 0: GPIO
set(PIO_USE_DMA   1)   # 1: use DMA, 0: not use DMA
set(I80_BUS_WR_CLK_KHZ 50000)
```

**4. 在src/CMakeLists.txth中添加新的显示屏驱动**
```cmake
set(LCD_DRV_USE_ILI9488 0)

...

if(LCD_DRV_USE_ILI9488)
    include(${CMAKE_CURRENT_LIST_DIR}/cmake/ili9488.cmake)
endif()

...

file(GLOB_RECURSE COMMON_SOURCES
    ...

    tft_ili9488.c

    ...
)

...

target_compile_definitions(${PROJECT_NAME} PUBLIC LCD_DRV_USE_ILI9488=${LCD_DRV_USE_ILI9488})
```

## 触摸屏

**1. 新建一个名为ft6236.c 的文件**
实现一个indev_spec结构体对象
```c
static struct indev_spec ft6236 = {
    .name = "ft6236",
    .type = INDEV_TYPE_POINTER,

    .i2c = {
        .addr = FT6236_ADDR,
        .master = i2c1,
        .speed = FT6236_DEF_SPEED,
        .pin_scl = FT6236_PIN_SCL,
        .pin_sda = FT6236_PIN_SDA,
    },

    .x_res = TOUCH_X_RES,
    .y_res = TOUCH_Y_RES,

    // .pin_irq = FT6236_PIN_IRQ,
    .pin_rst = FT6236_PIN_RST,

    .ops = {
        .write_reg  = ft6236_write_reg,
        .read_reg   = ft6236_read_reg,
        .init       = ft6236_hw_init,
        .is_pressed = ft6236_is_pressed,
        .read_x     = ft6236_read_x,
        .read_y     = ft6236_read_y,
    }
};

```