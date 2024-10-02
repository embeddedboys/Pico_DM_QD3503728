---
title: "embedded_graphics"
description: ""
summary: ""
date: 2024-10-03T00:38:17+08:00
lastmod: 2024-10-03T00:38:17+08:00
draft: false
weight: 1108
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

[rp-rs](https://github.com/rp-rs) 组织为RP2040开发了一个 Rust Embedded-HAL [rp-hal](https://github.com/rp-rs/rp-hal)，他们还提供了一个[工程模板](https://github.com/rp-rs/rp2040-project-template)，我们将在此工程基础上展开移植工作。

为了支持显示功能，我们需要在Cargo.toml中添加额外的crate:
```toml
embedded-graphics = "0.8.0"
embedded-graphics-core = "0.4.0"
display-interface-parallel-gpio = "0.7.0" # 提供数据接口，实现 WriteOnlyDataCommand trait
mipidsi = "0.8.0" # 提供 Display struct
```

在main.rs中，引用必要的crate:
```rust
// 使用兼容型号 ILI9486Rgb565，旋转等。
use mipidsi::{models::ILI9486Rgb565, options::Orientation, options::Rotation};
use mipidsi::options::ColorOrder;

// embedded_graphics 的一些接口等
use embedded_graphics::{
    mono_font::{ascii::FONT_10X20, MonoTextStyle},
    pixelcolor::Rgb888,
    prelude::*,
    primitives::{
        Circle, PrimitiveStyle, PrimitiveStyleBuilder, Rectangle, StrokeAlignment, Triangle,
    },
    text::{Alignment, Text},
};

// 提供 8080 16bit BUS GPIO 接口
use display_interface_parallel_gpio::{Generic16BitBus, PGPIO16BitInterface};

// 提供 Display struct
use mipidsi::Builder;
```

定义需要用到的 Pins
```rust
    let rst = pins.gpio22.into_push_pull_output_in_state(gpio::PinState::High);
    let wr = pins.gpio19.into_push_pull_output_in_state(gpio::PinState::High);
    let dc = pins.gpio20.into_push_pull_output();
    let _blk = pins.gpio28.into_push_pull_output_in_state(gpio::PinState::High);

    let lcd_d0 = pins.gpio0.into_push_pull_output();
    let lcd_d1 = pins.gpio1.into_push_pull_output();
    let lcd_d2 = pins.gpio2.into_push_pull_output();
    let lcd_d3 = pins.gpio3.into_push_pull_output();
    let lcd_d4 = pins.gpio4.into_push_pull_output();
    let lcd_d5 = pins.gpio5.into_push_pull_output();
    let lcd_d6 = pins.gpio6.into_push_pull_output();
    let lcd_d7 = pins.gpio7.into_push_pull_output();
    let lcd_d8 = pins.gpio8.into_push_pull_output();
    let lcd_d9 = pins.gpio9.into_push_pull_output();
    let lcd_d10 = pins.gpio10.into_push_pull_output();
    let lcd_d11 = pins.gpio11.into_push_pull_output();
    let lcd_d12 = pins.gpio12.into_push_pull_output();
    let lcd_d13 = pins.gpio13.into_push_pull_output();
    let lcd_d14 = pins.gpio14.into_push_pull_output();
    let lcd_d15 = pins.gpio15.into_push_pull_output();
```

创建 Display Interface，实现了 WriteOnlyDataCommand trait 的 struct
```rust
    let bus = Generic16BitBus::new((
        lcd_d0,
        lcd_d1,
        lcd_d2,
        lcd_d3,
        lcd_d4,
        lcd_d5,
        lcd_d6,
        lcd_d7,
        lcd_d8,
        lcd_d9,
        lcd_d10,
        lcd_d11,
        lcd_d12,
        lcd_d13,
        lcd_d14,
        lcd_d15,
    ));

    let di = PGPIO16BitInterface::new(bus, dc, wr);
```

定义旋转属性
```rust
let rotation = Orientation::new().rotate(Rotation::Deg270).flip_horizontal();
```

创建 Display struct, 并初始化，大部分显示库都是在此基础上绘制图形。
```rust
    let mut display = Builder::new(ILI9486Rgb565, di)
        .reset_pin(rst)
        .color_order(ColorOrder::Bgr)
        .orientation(rotation)
        .init(&mut delay)
        .unwrap();
```