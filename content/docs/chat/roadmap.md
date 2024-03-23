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

那么问题来了，如何在RP2040上面使用更大的内存呢？无独有偶，著名Hacker [Dmitry.GR](https://dmitry.gr/?r=01.Myself&proj=09.Personal)早就提出了一个解决方案——[ROMRAM](https://dmitry.gr/?r=06.%20Thoughts&proj=10.%20RomRam)，这个方案的大致原理就是利用RP2040本身的XIP来实现对SPI PSRAM的读写，问题是RP2040自带的XIP仅支持读取和执行，所以他使用了一个简单的门电路来控制对FLASH和PSRAM的互斥访问，然后通过MPU的Hardfault异常处理来进行PSRAM的写入，感兴趣的同学可以自行阅读他的那篇文章。 Dmitry的这个方案的问题在于外围电路复杂，且需要大量reg编程来配合使用。 这就有了后来的事情了。

更好的解决方案：
