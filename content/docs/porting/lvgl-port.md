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

如果您对LVGL十分感兴趣，我非常建议自己尝试一下移植LVGL，本文将以通俗易懂的方式，向大家介绍LVGL移植的过程，希望读者能在本文的辅助下，独自完成LVGL的移植。在本文的最后部分会向大家介绍有关性能优化的细节。

事实上，LVGL的每个大版本，driver部分的接口都有较大的变化，所以我们选择了两个目前最新的release版本讲述移植过程，分别是 `v8.3` 和 `v9`。

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

```c
static void ili9488_video_sync(struct ili9488_priv *priv, int xs, int ys, int xe, int ye, void *vmem16, size_t len)
{
    // pr_debug("video sync: xs=%d, ys=%d, xe=%d, ye=%d, len=%d\n", xs, ys, xe, ye, len);
    priv->tftops->set_addr_win(priv, xs, ys, xe, ye);
    write_buf_rs(priv, vmem16, len * 2, 1);
}
```

#### 全屏刷新

待添加

## LVGL 8.3

待添加

## LVGL 9

待添加

## 性能优化

待添加
