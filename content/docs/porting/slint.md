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

本拓展板的 Slint 移植工程：[https://github.com/embeddedboys/pico_dm_qd3503728_slint_mcu.git](https://github.com/embeddedboys/pico_dm_qd3503728_slint_mcu.git)

## 移植前的实验

在开始移植的时候，我们是基于微雪电子的 [Pico-ResTouch-LCD-2.8](https://www.waveshare.net/wiki/Pico-ResTouch-LCD-2.8) 来研究的，因为slint的官方MCU RP2040移植是基于此拓展板的。

下面这个链接是官方基于此拓展板编写的 Slint 打印机 demo 的代码。
[https://github.com/slint-ui/slint-mcu-rust-template](https://github.com/slint-ui/slint-mcu-rust-template)

这里还有一个Slint官方上传的演示视频。
[https://www.youtube.com/watch?v=dkBwNocItGs&t=1s](https://www.youtube.com/watch?v=dkBwNocItGs&t=1s)

微雪的这个拓展板总体上来说跟我们的拓展板差别还是非常大的，他们这个开发板是SPI驱动的ST7789，分辨率为320x240，使用XPT2046驱动电阻触摸屏。

第一版的时候我们使用两个现有的 crate 实现了一个简单的移植，可以参考这[embedded_graphics]中的内容。
```toml
display-interface-parallel-gpio = "0.7.0"
mipidsi = "0.8.0"
```

`display-interface-parallel-gpio` 这个库提供 I8080 协议的实现，因为是基于 GPIO 实现的，所以刷屏速度很慢。`mipidsi` 这个库提供 display 实现，属于显示驱动IC的层面。

## 移植到拓展板

移植到 slint 大致分为这么几个部分：

### 1. 创建一个基于 PIO 的显示 Bus，并实现 `WriteOnlyDataCommand` 这个 trait

> 你可以参考`pico_dm_qd3503728_slint_mcu/src/lib.rs` 中的实现。

这部分功能相当于 `display-interface-parallel-gpio` 这个 crate 所做的实现。

下面是 PIO 显示 Bus 的原型，以及一些将数据发送到 PIO 的函数

```rust
#[cfg(not(feature = "simulator"))]
pub struct Pio16BitBus<SM: ValidStateMachine, DC> {
    tx: Tx<SM>,
    dc: DC,
}

#[cfg(not(feature = "simulator"))]
impl<TX, DC> Pio16BitBus<TX, DC>
where
    TX: ValidStateMachine,
    DC: OutputPin,
{
    pub fn new(tx: Tx<TX>, dc: DC) -> Self {
        Self { tx, dc }
    }

    fn write_iter(&mut self, iter: impl Iterator<Item = u8>) -> Result {
        for value in iter {
            self.tx.write(value as u32);
            while !self.tx.is_empty() {}
        }
        Ok(())
    }

    fn write_iter16(&mut self, iter: impl Iterator<Item = u16>) -> Result {
        for value in iter {
            self.tx.write(value as u32);
            while !self.tx.is_empty() {}
        }
        Ok(())
    }

    pub fn write_data(&mut self, data: DataFormat<'_>) -> Result {
        match data {
            DataFormat::U8(slice) => self.write_iter(slice.iter().copied()),
            DataFormat::U16LEIter(iter) => self.write_iter16(iter),
            _ => Err(DisplayError::DataFormatNotImplemented),
        }
    }
}
```

实现 `WriteOnlyDataCommand` 这个 trait 的代码如下：

```rust
#[cfg(not(feature = "simulator"))]
impl<TX, DC> WriteOnlyDataCommand for Pio16BitBus<TX, DC>
where
    TX: ValidStateMachine,
    DC: OutputPin,
{
    fn send_commands(&mut self, cmd: DataFormat<'_>) -> Result {
        self.dc.set_low().map_err(|_| DisplayError::DCError)?;
        self.write_data(cmd)?;
        Ok(())
    }
    fn send_data(&mut self, buf: DataFormat<'_>) -> Result {
        self.dc.set_high().map_err(|_| DisplayError::DCError)?;
        self.write_data(buf)?;
        Ok(())
    }
}
```

`pico_dm_qd3503728_slint_mcu/src/main.rs` 中有创建 PIO 对象，并注册进 PIO bus 的代码：

```rust
    let program = pio_proc::pio_asm!(
        ".side_set 1"
        ".wrap_target",
        "   out pins, 16    side 0",
        "   nop             side 1",
        ".wrap"
    );

    let wr: Pin<_, FunctionPio0, _> = pins.gpio19.into_function();
    let wr_pin_id = wr.id().num;

    let dc = pins.gpio20.into_push_pull_output();
    let rst = pins.gpio22.into_push_pull_output();
    let bl = pins.gpio28.into_push_pull_output();

    let lcd_d0: Pin<_, FunctionPio0, _> = pins.gpio0.into_function();
    let lcd_d1: Pin<_, FunctionPio0, _> = pins.gpio1.into_function();
    let lcd_d2: Pin<_, FunctionPio0, _> = pins.gpio2.into_function();
    let lcd_d3: Pin<_, FunctionPio0, _> = pins.gpio3.into_function();
    let lcd_d4: Pin<_, FunctionPio0, _> = pins.gpio4.into_function();
    let lcd_d5: Pin<_, FunctionPio0, _> = pins.gpio5.into_function();
    let lcd_d6: Pin<_, FunctionPio0, _> = pins.gpio6.into_function();
    let lcd_d7: Pin<_, FunctionPio0, _> = pins.gpio7.into_function();
    let lcd_d8: Pin<_, FunctionPio0, _> = pins.gpio8.into_function();
    let lcd_d9: Pin<_, FunctionPio0, _> = pins.gpio9.into_function();
    let lcd_d10: Pin<_, FunctionPio0, _> = pins.gpio10.into_function();
    let lcd_d11: Pin<_, FunctionPio0, _> = pins.gpio11.into_function();
    let lcd_d12: Pin<_, FunctionPio0, _> = pins.gpio12.into_function();
    let lcd_d13: Pin<_, FunctionPio0, _> = pins.gpio13.into_function();
    let lcd_d14: Pin<_, FunctionPio0, _> = pins.gpio14.into_function();
    let lcd_d15: Pin<_, FunctionPio0, _> = pins.gpio15.into_function();

    let lcd_d0_pin_id = lcd_d0.id().num;

    let pindirs = [
        (wr_pin_id, hal::pio::PinDir::Output),
        (lcd_d0.id().num, hal::pio::PinDir::Output),
        (lcd_d1.id().num, hal::pio::PinDir::Output),
        (lcd_d2.id().num, hal::pio::PinDir::Output),
        (lcd_d3.id().num, hal::pio::PinDir::Output),
        (lcd_d4.id().num, hal::pio::PinDir::Output),
        (lcd_d5.id().num, hal::pio::PinDir::Output),
        (lcd_d6.id().num, hal::pio::PinDir::Output),
        (lcd_d7.id().num, hal::pio::PinDir::Output),
        (lcd_d8.id().num, hal::pio::PinDir::Output),
        (lcd_d9.id().num, hal::pio::PinDir::Output),
        (lcd_d10.id().num, hal::pio::PinDir::Output),
        (lcd_d11.id().num, hal::pio::PinDir::Output),
        (lcd_d12.id().num, hal::pio::PinDir::Output),
        (lcd_d13.id().num, hal::pio::PinDir::Output),
        (lcd_d14.id().num, hal::pio::PinDir::Output),
        (lcd_d15.id().num, hal::pio::PinDir::Output),
    ];

    let (mut pio, sm0, _, _, _) = pac.PIO0.split(&mut pac.RESETS);
    let installed = pio.install(&program.program).unwrap();
    let (int, frac) = (1, 0); // as slow as possible (0 is interpreted as 65536)
    let (mut sm, _, tx) = rp2040_hal::pio::PIOBuilder::from_installed_program(installed)
        .side_set_pin_base(wr_pin_id)
        .out_pins(lcd_d0_pin_id, 16)
        .buffers(Buffers::OnlyTx)
        .clock_divisor_fixed_point(int, frac)
        .out_shift_direction(ShiftDirection::Right)
        .autopull(true)
        .pull_threshold(16)
        .build(sm0);
    sm.set_pindirs(pindirs);
    sm.start();

    info!("PIO block setuped");

    let di = Pio16BitBus::new(tx, dc);
```

### 2. 创建 ILI9488 驱动IC 的结构体

它基于 `WriteOnlyDataCommand` 这个 trait 发送 Data 和 Command 给 ILI9488 驱动IC。

这是 ILI9488 驱动IC 的结构体及其相关函数实现，包括复位、发送初始化序列、设置绘制区域等：

```rust
#[cfg(not(feature = "simulator"))]
pub struct ILI9488<DI, RST, BL>
where
    DI: WriteOnlyDataCommand,
    RST: OutputPin,
    BL: OutputPin,
{
    di: DI,
    rst: Option<RST>,
    bl: Option<BL>,

    size_x: u16,
    size_y: u16,
}

#[cfg(not(feature = "simulator"))]
impl<DI, RST, BL> ILI9488<DI, RST, BL>
where
    DI: WriteOnlyDataCommand,
    RST: OutputPin,
    BL: OutputPin,
{
    pub fn new(di: DI, rst: Option<RST>, bl: Option<BL>, size_x: u16, size_y: u16) -> Self {
        Self {
            di,
            rst,
            bl,
            size_x,
            size_y,
        }
    }

    pub fn init_test(&mut self) -> Result {
        self.write_command(0x55)?;
        Ok(())
    }

    pub fn init(&mut self, delay_source: &mut Delay) -> Result {
        self.hard_reset(delay_source);

        if let Some(bl) = self.bl.as_mut() {
            bl.set_high().unwrap();
        }

        // Positive Gamma Control
        self.write_reg(&[0xE0, 0x00, 0x03, 0x09, 0x08, 0x16, 0x0A, 0x3F, 0x78, 0x4C, 0x09, 0x0A, 0x08, 0x16, 0x1A, 0x0F])?;

        // Negative Gamma Control
        self.write_reg(&[0xE1, 0x00, 0x16, 0x19, 0x03, 0x0F, 0x05, 0x32, 0x45, 0x46, 0x04, 0x0E, 0x0D, 0x35, 0x37, 0x0F])?;

        self.write_reg(&[0xC0, 0x17, 0x15])?;          // Power Control 1
        self.write_reg(&[0xC1, 0x41])?;                // Power Control 2
        self.write_reg(&[0xC5, 0x00, 0x12, 0x80])?;    // VCOM Control
        self.write_reg(&[0x36, 0x28])?;                // Memory Access Control
        self.write_reg(&[0x3A, 0x55])?;                // Pixel Interface Format RGB565 8080 16-bit
        self.write_reg(&[0xB0, 0x00])?;                // Interface Mode Control

        // Frame Rate Control
        // self.write_reg(&[0xB1, 0xD0, 0x11);              // 60Hz
        self.write_reg(&[0xB1, 0xD0, 0x14])?;          // 90Hz

        self.write_reg(&[0xB4, 0x02])?;                // Display Inversion Control
        self.write_reg(&[0xB6, 0x02, 0x02, 0x3B])?;    // Display Function Control
        self.write_reg(&[0xB7, 0xC6])?;                // Entry Mode Set
        self.write_reg(&[0xF7, 0xA9, 0x51, 0x2C, 0x82])?;  // Adjust Control 3
        self.write_reg(&[0x11])?;                      // Exit Sleep
        delay_source.delay_ms(60);
        self.write_reg(&[0x29])?;                      // Display on

        Ok(())
    }

    pub fn hard_reset(&mut self, delay_source: &mut Delay) {
        if let Some(rst) = self.rst.as_mut() {
            rst.set_high().unwrap();
            delay_source.delay_ms(10);
            rst.set_low().unwrap();
            delay_source.delay_ms(10);
            rst.set_high().unwrap();
            delay_source.delay_ms(10);
        }
    }

    pub fn set_addr_win(&mut self, xs: u16, ys: u16, xe: u16, ye: u16) -> Result {
        self.write_reg(&[
            0x2A,
            (xs >> 8) as u8,
            (xs) as u8,
            (xe >> 8) as u8,
            (xe) as u8,
        ])?;

        self.write_reg(&[
            0x2B,
            (ys >> 8) as u8,
            (ys) as u8,
            (ye >> 8) as u8,
            (ye) as u8,
        ])?;

        self.write_reg(&[0x2C])?;
        Ok(())
    }

    pub fn write_pixels<I>(&mut self, colors: I) -> Result
    where
        I: IntoIterator<Item = Rgb565>,
    {
        let mut iter = colors.into_iter().map(|c| c.into_storage());
        let buf = DataFormat::U16LEIter(&mut iter);
        self.di.send_data(buf)?;
        Ok(())
    }

    pub fn write_command(&mut self, cmd: u8) -> Result {
        self.di.send_commands(DataFormat::U8(&[cmd]))?;
        Ok(())
    }

    pub fn write_data(&mut self, buf: u8) -> Result {
        self.di.send_data(DataFormat::U8(&[buf]))?;
        Ok(())
    }

    pub fn write_reg(&mut self, seq: &[u8]) -> Result {
        self.write_command(seq[0])?;
        for val in &seq[1..] {
            self.write_data(*val)?;
        }
        Ok(())
    }
}
```

在 `main.rs` 中创建 ILI9488 驱动IC 的实例，并调用显示初始化接口：

```rust
  let mut display = ILI9488::new(di, Some(rst), Some(bl), 480, 320);
  display.init(&mut delay).unwrap();
```

### 3. 为 ILI9488 驱动IC 结构体实现 `DrawTarget` 和 `OriginDimensions` 这两个trait

你可以参考 `pico_dm_qd3503728_slint_mcu/src/graphics.rs` 这个文件中的实现

`DrawTarget` 的实现：

```rust
#[cfg(not(feature = "simulator"))]
impl<DI, RST, BL> DrawTarget for ILI9488<DI, RST, BL>
where
    DI: WriteOnlyDataCommand,
    RST: OutputPin,
    BL: OutputPin,
{
    type Color = Rgb565;
    type Error = DisplayError;

    fn draw_iter<I>(&mut self, pixels: I) -> Result<(), Self::Error>
    where
        I: IntoIterator<Item = Pixel<Self::Color>>,
    {
        for pixel in pixels {
            let x = pixel.0.x as u16;
            let y = pixel.0.y as u16;

            self.set_addr_win(x, y, x, y)?;
            self.write_pixels([pixel.1])?;
        }
        Ok(())
    }

    fn fill_contiguous<I>(&mut self, area: &Rectangle, colors: I) -> Result<(), Self::Error>
    where
        I: IntoIterator<Item = Self::Color>,
    {
        let intersection = area.intersection(&self.bounding_box());
        let Some(bottom_right) = intersection.bottom_right() else {
            // No intersection -> nothing to draw
            return Ok(());
        };
        let xs = area.top_left.x as u16;
        let ys = area.top_left.y as u16;
        let xe = bottom_right.x as u16;
        let ye = bottom_right.y as u16;

        let colors = colors.into_iter();
        if &intersection == area {
            // Draw the original iterator if no edge overlaps the framebuffer
            self.set_addr_win(xs, ys, xe, ye)?;
            self.write_pixels(colors)?;
        } else {
            info!("overlap not supported yet");
        }
        Ok(())
    }

    fn fill_solid(&mut self, area: &Rectangle, color: Self::Color) -> Result<(), Self::Error> {
        let Some(bottom_right) = area.bottom_right() else {
            // No intersection -> nothing to draw
            return Ok(());
        };
        let xs = area.top_left.x as u16;
        let ys = area.top_left.y as u16;
        let xe = bottom_right.x as u16;
        let ye = bottom_right.y as u16;

        let count = area.size.width * area.size.height;
        let colors = core::iter::repeat(color).take(count.try_into().unwrap());

        self.set_addr_win(xs, ys, xe, ye)?;
        self.write_pixels(colors)?;
        Ok(())
    }
}
```

`OriginDimensions` 的实现：

```rust
#[cfg(not(feature = "simulator"))]
impl<DI, RST, BL> OriginDimensions for ILI9488<DI, RST, BL>
where
    DI: WriteOnlyDataCommand,
    RST: OutputPin,
    BL: OutputPin,
{
    fn size(&self) -> Size {
        Size::new(self.size_x as u32, self.size_y as u32)
    }
}
```