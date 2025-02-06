---
title: "说明"
description: ""
summary: ""
date: 2024-03-27T22:46:40Z
lastmod: 2024-03-27T22:46:40Z
draft: false
weight: 1101
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

本章节将会讲解本产品开发过程中涉及的移植问题，一方面是为了向用户公开更多的资料，另一方面也是打算将本章节作为相关资料的wiki，方便我们以后查阅。

如果用户对这方面比较感兴趣，可以仔细阅读本章节，并通过完成若干个实验来学习了解这个开发过程。

为了更好的推广本产品，我们决定将本产品移植适配其他平台，例如Linux、ESP32等。

因为受限于本产品接口，所以仅支持兼容树莓派Pico引脚定义的开发板，
目前市面上能找到的类似开发板不多，如果列表中的没有你所使用的型号，
可以联系我们有偿提供支持。

请根据情况，在本章节寻找适合您的平台——开发板对应的文章。

首批提上日程的开发板如下：

### Linux

- [x] [幸狐科技 Luckfox Pico](http://localhost:1313/docs/porting/linux/#luckfox_pico)
- [x] [幸狐科技 Luckfox Pico Max](http://localhost:1313/docs/porting/linux/#luckfox_pico)
- [ ] [幸狐科技 Luckfox Lyra Plus]()
- [x] [Milk-V Duo](http://localhost:1313/docs/porting/linux/#milk-v-duo)
- [ ] [Milk-V Duo 256M](http://localhost:1313/docs/porting/linux/#milk-v-duo-256M)

### ESP32-S3
- [ ] [WalnutPi 核桃派PicoW ESP32-S3]()
- [x] [无名科技 ESP32-S3 Pico]()
- [x] [Unknown ESP32-S3 Dev Board A]()

{{< callout context="note" title="说明" icon="info-circle" >}}
部分开发板存在缺少引脚引出的情况，所以可能会需要飞线。
若有需要，则会在对应文章中指出。
{{< /callout >}}
