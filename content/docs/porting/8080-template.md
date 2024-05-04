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

**3. 在src/CMakeLists.txth中添加新的显示屏驱动**
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