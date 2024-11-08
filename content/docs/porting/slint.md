---
title: "Slint"
description: ""
summary: ""
date: 2024-05-04T22:52:10+08:00
lastmod: 2024-05-04T22:52:10+08:00
draft: false
weight: 1109
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

Slint 官网：[https://slint.dev/](https://slint.dev/)

在开始移植的时候，我们是基于微雪电子的 [Pico-ResTouch-LCD-2.8](https://www.waveshare.net/wiki/Pico-ResTouch-LCD-2.8) 来研究的，因为slint的官方MCU RP2040移植是基于此拓展板的。

下面这个链接是官方基于此拓展板编写的 Slint 打印机 demo 的代码。
[https://github.com/slint-ui/slint-mcu-rust-template](https://github.com/slint-ui/slint-mcu-rust-template)

这里还有一个Slint官方上传的演示视频。
[https://www.youtube.com/watch?v=dkBwNocItGs&t=1s](https://www.youtube.com/watch?v=dkBwNocItGs&t=1s)

微雪的这个拓展板总体上来说跟我们的拓展板差别还是非常大的，他们这个开发板是SPI驱动的ST7789，分辨率为320x240，使用XPT2046驱动电阻触摸屏。

第一版的时候我们的开发人员不清楚这部分接口的实现，所以使用下面这两个现成的 crate 实现了一个简单的移植
```toml
display-interface-parallel-gpio = "0.7.0"
mipidsi = "0.8.0"
```

`display-interface-parallel-gpio` 这个 crate 是基于 GPIO 的，所以刷新速度非常的慢，`mipidsi` 这个库提供 display 实现，属于显示驱动IC的层面。 这两个 crate 的作用可以看 [embedded_graphics] 中的介绍。