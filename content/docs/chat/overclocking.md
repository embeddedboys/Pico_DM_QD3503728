---
title: "超频"
description: ""
summary: ""
date: 2024-04-22T23:02:04+08:00
lastmod: 2024-04-22T23:02:04+08:00
draft: false
weight: 1004
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

树梅派RP2040的额定最大工作频率为133MHz，然而其最大潜力原不止于此，适当的超频可以让MCU提升一部分性能，当然是在可接受的范围内，所以我们觉得还是有必要单独开一个章节来讨论超频的问题。

(在此处引用rp2040超频至1GHz的文章)

(在此处引用rp2040超频测试的视频)

## 合理超频

## 极限超频

### 调整DVDD电压
如果我们想让CPU运行在更高的频率，加压往往是最简单的方式，但是随之而来的问题是额外的功耗和发热量。

pico-sdk提供了一套接口，用来调整DVDD的电压，从0.9V到1.3V不等多个梯度。经过我们多方面测试，在1.3V电压下，能成功启动的最大频率为:

- CPU : 436MHz
- Flash : 109MHz

如果您还想继续提升频率，就需要再继续提高DVDD电压了，可以通过飞线解决，我们使用一个外部的可调电源来提供DVDD电压。

（飞线示意图）

最开始提到的文章中，作者超频至1GHz时，DVDD电压为3V，我们不需要这么高的频率，实际上，我们只需要`133 X 4 = 532MHz`就够了，设置Flash QSPI分频系数为4，这样满足了Flash的最大工作频率，如果设置为8分频，则需要`133 X 8 = 1064MHz`的CPU频率，这需要额外的冷却措施，太麻烦，所以`532MHz`很好，我们只需要测试多少DVDD电压能稳定在这个频率就可以了。

### 更换更高性能的Flash芯片
```CPU运行在这么高的频率，我们的Flash老兄都快受不了了```

在如此高的工作频率下，Flash芯片也会带来更多的发热量，如果其质量低劣，很可能已经罢工了，但是等待冷却后，又会恢复正常，这是我们观察到的现象。
