---
title: "LVGL移植"
description: ""
summary: ""
date: 2024-03-23T01:02:33Z
lastmod: 2024-03-23T01:02:33Z
draft: false
weight: 1104
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 写在前面

如果您对LVGL十分感兴趣，我非常建议自己尝试一下移植LVGL，本文将以通俗易懂的方式，向大家介绍LVGL移植的过程，希望读者能在本文的辅助下，独自完成LVGL的移植。 在本文的最后部分会向大家介绍有关性能优化的细节。

事实上，LVGL的每个大版本，driver部分的接口都有较大的变化，所以我们选择了两个目前最新的release版本讲述移植过程，分别是 `v8.4` 和 `v9`。

**你可以在 `lvgl` 工程源码的 `examples/porting` 文件夹下找到当前版本的移植模板文件**

## 准备工作

如果你想要在你的显示模组上移植LVGL，最先应该做的是将其点亮。 比如实现一个描点函数。

为了验证屏幕处于可用状态，我们通常会在HAL层实现一些基本功能，比如复位、初始化、通常分为以下几步：

### 1. 复位显示模组

一般来说，大部分显示面板都提供了 RESET 的复位引脚，且低有效。
通常这是在向显示模组写入初始化序列的前一步。

```c
#define dm_gpio_set_value(p,v) gpio_put(p, v)
#define mdelay(v) sleep_ms(v)
static int ili9488_reset(struct ili9488_priv *priv)
{
    dm_gpio_set_value(priv->gpio.reset, 1);
    mdelay(10);
    dm_gpio_set_value(priv->gpio.reset, 0);
    mdelay(10);
    dm_gpio_set_value(priv->gpio.reset, 1);
    mdelay(10);
    return 0;
}
```


### 2. 向显示模组写入初始化序列

所谓初始化序列，其实就是命令 + 数据的组合，举个例子：

```c
write_reg(priv, 0x3A, 0x55);
```

代表发送 `触发像素格式设置命令` + `设置像素格式为RGB565`

这一系列设置命令完成了对显示驱动IC的初步设置。

```c
static int ili9488_init_display(struct ili9488_priv *priv)
{
    pr_debug("%s, writing initial sequence...\n", __func__);
    ili9488_reset(priv);
    // dm_gpio_set_value(&priv->gpio.rd, 1);
    // mdelay(150);

    // Positive Gamma Control
    write_reg(priv, 0xE0, 0x00, 0x03, 0x09, 0x08, 0x16, 0x0A, 0x3F, 0x78, 0x4C, 0x09, 0x0A, 0x08, 0x16, 0x1A, 0x0F);

    // Negative Gamma Control
    write_reg(priv, 0xE1, 0x00, 0x16, 0x19, 0x03, 0x0F, 0x05, 0x32, 0x45, 0x46, 0x04, 0x0E, 0x0D, 0x35, 0x37, 0x0F);

    write_reg(priv, 0xC0, 0x17, 0x15);          // Power Control 1
    write_reg(priv, 0xC1, 0x41);                // Power Control 2
    write_reg(priv, 0xC5, 0x00, 0x12, 0x80);    // VCOM Control
    // write_reg(priv, 0x36, 0x28);                // Memory Access Control
    write_reg(priv, 0x3A, 0x55);                // Pixel Interface Format RGB565 8080 16-bit
    write_reg(priv, 0xB0, 0x00);                // Interface Mode Control

    // Frame Rate Control
    // write_reg(priv, 0xB1, 0xD0, 0x11);       // 60Hz
    write_reg(priv, 0xB1, 0xD0, 0x14);          // 90Hz

    write_reg(priv, 0xB4, 0x02);                // Display Inversion Control
    write_reg(priv, 0xB6, 0x02, 0x02, 0x3B);    // Display Function Control
    write_reg(priv, 0xB7, 0xC6);                // Entry Mode Set
    write_reg(priv, 0xF7, 0xA9, 0x51, 0x2C, 0x82);  // Adjust Control 3
    write_reg(priv, 0x11);                      // Exit Sleep
    mdelay(60);
    write_reg(priv, 0x29);                      // Display on

    return 0;
}
```

### 3. 设置绘制区域

一些显示驱动IC，为了节省MCU的内存，将显示buffer集成到了显示驱动IC中。 这样做的
另一个优势是不强制要求MCU侧具备强大的显示控制器，MCU在讲数据写入到显示驱动IC的SRAM
时，驱动IC会自行将SRAM中的数据刷新到显示面板上。

所以像这种驱动IC，基本都支持局部刷新，我们只需要将显示缓冲区中需要更新的部分写入到SRAM中即可，
这类驱动IC一般都提供了一个设置绘制区域的命令，例如 `0x2A (设置列地址)` `0x2B (设置行地址)`，
在发送完设置命令之后，还需要发送具体参数，请参考数据手册中的要求。

下面这个接口展示了通过 `xs`, `ys`, `xe`, `ye` 四个参数，设置一个从 (xs, ys) 到 (xe, ye) 的矩形刷新区域。

```c
static int ili9488_set_addr_win(struct ili9488_priv *priv, int xs, int ys, int xe,
                                int ye)
{
    /* set column adddress */
    write_reg(priv, 0x2A, xs >> 8, xs, xe >> 8, xe);

    /* set row address */
    write_reg(priv, 0x2B, ys >> 8, ys, ye >> 8, ye);

    /* write start */
    write_reg(priv, 0x2C);
    return 0;
}
```

`0x2C` 命令用于通知驱动IC将要写入SRAM。

### 4. 发送颜色数据

#### 局部刷新

发送颜色数据一般跟在 `0x2C` 命令之后，如果你选择局部数据更新，则还应该在这之前设置绘制区域 `0x2A`, `0x2B`。

如下所示的是一个窗口绘制函数，接受一个矩形区域的参数和一个给定长度的像素数据buffer

```c
static void ili9488_video_sync(struct ili9488_priv *priv, int xs, int ys, int xe, int ye, void *vmem16, size_t len)
{
    // pr_debug("video sync: xs=%d, ys=%d, xe=%d, ye=%d, len=%d\n", xs, ys, xe, ye, len);
    priv->tftops->set_addr_win(priv, xs, ys, xe, ye);
    write_buf_rs(priv, vmem16, len * 2, 1);
}
```

在本工程中，`write_buf_rs` 是一个宏定义，会根据另一个宏`DISP_OVER_PIO`来决定是否通过`PIO`外设模拟I8080协议发送数据

```c
/* rs=0 means writing register, rs=1 means writing data */
#if DISP_OVER_PIO
    #define write_buf_rs(p, b, l, r) i80_write_buf_rs(b, l, r)
#else
    #define write_buf_rs(p, b, l, r) fbtft_write_gpio16_wr_rs(p, b, l, r)
#endif
```

下面是两个函数的原型，执行的逻辑是先设置RS引脚，再发送给定buffer中的数据到I8080端口。

```c
static void fbtft_write_gpio16_wr_rs(struct ili9488_priv *priv, void *buf, size_t len, bool rs)
void i80_write_buf_rs(void *buf, size_t len, bool rs);
```

#### 全屏刷新

如果你需要全屏刷新，则必须先分配一个全屏的buffer，在本工程中，一块全屏buffer的大小为 `307200` 字节，
这超出了RP2040上的 256KB SRAM 限制，不过在RP2350上，有512KB SRAM，可以使用全屏刷新。

在发送完初始化序列后，设置一次绘制区域为整屏大小，然后发送命令`0x2C`，下面是一个示例

```
priv->tftops->set_addr_win(priv, 0, 0,
                         priv->display->xres - 1,
                         priv->display->yres - 1);
```

然后就可以循环发送全屏buffer，驱动IC中的行列指针会根据发送的数据自动移动的复位。

全屏刷新一般用于内部有 LCD 控制器的 MCU 或 MPU，例如在 F1C200s 上，可以将 TCON 设置成I8080模式，
在初始化TCON之前，先通过GPIO模拟I8080端口，初始化显示屏，然后再初始化 TCON, BE, FE 等，根据framebuffer
中的数据输出I8080时序，就可以实现 LCD 控制器全屏刷新的效果。


## LVGL 8.4

### 显示驱动

可以参考`pico_dm_qd3503728_noos/porting/lv_port_disp_template.c`

#### 1. 分配 LVGL 显示 buffer
```c

    /* 你可以在这里调用 LCD 的初始化函数 */
    disp_init();

#ifndef MY_DISP_BUF_SIZE
    #warning '"MY_DISP_BUF_SIZE" is not defined, defaulting to (HOR_RES * VER_RES / 2)'
    #define MY_DISP_BUF_SIZE    (MY_DISP_HOR_RES * MY_DISP_VER_RES / 2)
#endif

    static lv_disp_draw_buf_t draw_buf_dsc_1;
    static lv_color_t buf_1[MY_DISP_BUF_SIZE];

    /* 调用lvgl接口初始化显示buffer */
    lv_disp_draw_buf_init(&draw_buf_dsc_1, buf_1, NULL, MY_DISP_BUF_SIZE);
```

#### 2. 分配、注册 LVGL 显示驱动
```c
    static lv_disp_drv_t disp_drv;                         /*Descriptor of a display driver*/
    lv_disp_drv_init(&disp_drv);                    /*Basic initialization*/

    /* 你的屏幕的宽高分辨率 */
    disp_drv.hor_res = MY_DISP_HOR_RES;
    disp_drv.ver_res = MY_DISP_VER_RES;

    /* 这个是需要你实现的显示 buffer 刷新的回调函数 */
    disp_drv.flush_cb = ili9488_flush;

    /* 将第一步分配的 LVGL 显示 buffer 注册到显示驱动中 */
    disp_drv.draw_buf = &draw_buf_dsc_1;

    /* 注册显示驱动 */
    lv_disp_drv_register(&disp_drv);
```

下面是 `ili9488_flush` 相关函数的实现

```c
static inline void ili9488_write_cmd(uint16_t cmd)
{
    write_buf_rs(&g_priv, &cmd, sizeof(cmd), 0);
}
#define write_cmd ili9488_write_cmd
static inline void ili9488_write_data(uint16_t data)
{
    write_buf_rs(&g_priv, &data, sizeof(data), 1);
}
#define write_data ili9488_write_data

#include "lvgl/lvgl.h"
void ili9488_flush(lv_disp_drv_t * disp_drv, const lv_area_t * area, lv_color_t * color_p)
{
#if 1
    write_cmd(0x2A);
    write_data(area->x1 >> 8);
    write_data(area->x1);
    write_data(area->x2 >> 8);
    write_data(area->x2);

    /* set row address */
    write_cmd(0x2B);
    write_data(area->y1 >> 8);
    write_data(area->y1);
    write_data(area->y2 >> 8);
    write_data(area->y2);

    /* write start */
    write_cmd(0x2C);
    write_buf_rs(&g_priv, (void *)color_p, lv_area_get_size(area) * 2, 1);
#else
    struct ili9488_priv *priv = &g_priv;
    priv->tftops->set_addr_win(priv, area->x1, area->y1, area->x2, area->y2);
    write_buf_rs(priv, (void *)px_map, lv_area_get_size(area) * 2, 1);
#endif
    lv_disp_flush_ready(disp_drv);
}
```

### 输入驱动

可以参考`pico_dm_qd3503728_noos/porting/lv_port_indev_template.c`

#### 1. 分配、注册 LVGL Touchpad 驱动

```c
    static lv_indev_drv_t indev_drv;

    /*------------------
     * Touchpad
     * -----------------*/

    /* 你可以在这里初始化你的触摸屏 */
    touchpad_init();

    /* 初始化 LVGL 触摸屏驱动 */
    lv_indev_drv_init(&indev_drv);

    /* 设置驱动类型为点类型 */
    indev_drv.type = LV_INDEV_TYPE_POINTER;

    /* 设置触摸屏驱动的读取回调函数，这个需要你自己实现 */
    indev_drv.read_cb = touchpad_read;

    /* 注册触摸屏驱动 */
    indev_touchpad = lv_indev_drv_register(&indev_drv);
```

下面是 `touchpad_read` 相关函数的实现

```c
static void touchpad_read(lv_indev_drv_t * indev_drv, lv_indev_data_t * data)
{
    static lv_coord_t last_x = 0;
    static lv_coord_t last_y = 0;

    /*Save the pressed coordinates and the state*/
    if(touchpad_is_pressed()) {
        touchpad_get_xy(&last_x, &last_y);
        // printf("touchpad is pressed, x: %d, y: %d\n", last_x, last_y);
        data->state = LV_INDEV_STATE_PR;
    }
    else {
        data->state = LV_INDEV_STATE_REL;
    }

    /*Set the last pressed coordinates*/
    data->point.x = last_x;
    data->point.y = last_y;
}

static bool touchpad_is_pressed(void)
{
    /*Your code comes here*/
    return ft6236_is_pressed();
    // return false;
}

static void touchpad_get_xy(lv_coord_t * x, lv_coord_t * y)
{
    /*Your code comes here*/
    (*x) = ft6236_read_x();
    (*y) = ft6236_read_y();
}
```

### Tick 驱动

v8.4 版本的 LVGL 允许在 lv_conf.h 中设置 tick 回调函数

```c
#define LV_TICK_CUSTOM 1
#if LV_TICK_CUSTOM
    #define LV_TICK_CUSTOM_INCLUDE "pico/time.h"         /*Header for the system time function*/
    #define LV_TICK_CUSTOM_SYS_TIME_EXPR (time_us_32() / 1000LL)    /*Expression evaluating to current system time in ms*/
    /*If using lvgl as ESP32 component*/
    // #define LV_TICK_CUSTOM_INCLUDE "esp_timer.h"
    // #define LV_TICK_CUSTOM_SYS_TIME_EXPR ((esp_timer_get_time() / 1000LL))
#endif   /*LV_TICK_CUSTOM*/
```

`time_us_32()` 是 Pico SDK 中的一个函数，用于获取系统启动后经过的时间，单位为微秒，因为 LVGL 的tick以毫秒为单位，所以需要除1000

## LVGL 9

待添加

## 性能优化

待添加
