---
title: "实验性工作"
description: ""
summary: ""
date: 2024-03-23T01:01:24Z
lastmod: 2024-03-23T01:01:24Z
draft: false
weight: 1003
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

我们在移植LVGL到RP2040的过程中，其实遇到了很多的问题，比如默认频率跑的太慢了、如何利用两个核心工作等等，其中大部分难题都得到了解决，唯独内存太小这个问题现在依然没有解决，我想这也是所有类似MCU的通病了。

我们都知道，RP2040上可用的内存有264KB，这是算上了两块其他外设的4KB SRAM，实际上只有256KB能用，所以我们来做一下数学，一块480x320 16bpp的屏幕，一帧画面为300KB，这已经超出了256KB，更别忘了还有.data等放在sram中的数据段。

```shell
使用了1/2屏幕大小lvgl buffer后的内存占用情况

Memory region         Used Size  Region Size  %age Used
           FLASH:      349560 B         2 MB     16.67%
             RAM:      230364 B       256 KB     87.88%
       SCRATCH_X:          0 GB         4 KB      0.00%
       SCRATCH_Y:          0 GB         4 KB      0.00%
```

所以一般情况下，我们只能申请半块屏幕大小的Buffer来供LVGL使用。 原来需要一次就能完成的工作，现在需要两次或多次，这就产生了刷新率下降的问题。

那么问题来了，如何在RP2040上面使用更大的内存呢？无独有偶，著名Hacker [Dmitry.GR](https://dmitry.gr/?r=01.Myself&proj=09.Personal)早就提出了一个解决方案——[ROMRAM](https://dmitry.gr/?r=06.%20Thoughts&proj=10.%20RomRam)，这个方案的大致原理就是利用RP2040本身的XIP来实现对SPI PSRAM的读写，问题是RP2040自带的XIP仅支持读取和执行，所以他使用了一个简单的门电路来控制对FLASH和PSRAM的互斥访问，然后通过MPU的Hardfault异常处理来进行PSRAM的写入，这就使得PSRAM看起来像**memory mapped**一样了，感兴趣的同学可以自行阅读他的那篇文章。 Dmitry的这个方案的问题在于外围电路复杂，且需要大量register编程来配合使用。 所以这就有了后来的事情了。

我在翻阅树莓派论坛的时候，发现了另一篇关于RP2040 PSRAM的文章——[
Using SPI PSRAM with the RP2040](https://forums.raspberrypi.com/viewtopic.php?t=316012)，这篇文章作者提出的方法更为简单，他的大致方法是这样的：

1. 使用MPU来保护一段memory总线上未映射的内存

例如 `0x2F000000` - `0x2F002000` 8MB 大小的内存，当然也可以是其他地方的

2. 使用指针来尝试访问受MPU限制的内存区域，触发Hardfault

3. 在HardFault异常处理函数（isr_hardfault）中:

    3.1 存储当前r0等寄存器状态，类似于OS context switch，因为我们要调用其他函数

    3.2 设置参数，并调用c handler函数

    3.3 handler函数中解析非法指令，完成该指令试图的操作（对非法内存区域的读写）

    3.4 恢复寄存器状态，并通过LR寄存器返回异常发生处。

4. 程序继续执行

问题是，如果每次在handler函数中都对psram进行访问，这导致了频繁的外设访问，间接的降低了内存访问速度，我们可以引入高速cache的概念，在两者之前加入一个简单有效的cache，可以缓解这个问题。 但是我没有进行实验。

所以我根据上述资料写了一个简单的hardfault访问一个测试数组的demo，位于[psram-mpu](https://github.com/IotaHydrae/rpi-pico-lab/blob/main/psram-mpu/main.c)，这个示例完成了在handler函数中对fake_memory数组的写入，将其替换为对psram的访问，并加入cache机制应该就能达到刚刚说的效果。 

由于最近在忙于其他项目，这个实验性的工作我没有投入太多时间，所以，先这样吧。