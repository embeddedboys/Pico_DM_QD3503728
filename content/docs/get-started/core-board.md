---
title: "核心板的选择（必读）"
description: ""
summary: ""
date: 2024-03-12T10:10:16Z
lastmod: 2024-03-12T10:10:16Z
draft: false
weight: 103
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 说明

我们默认不提供核心板，主要有如下几个原因：

- 核心板利润过低，市场均价10元左右
- 用户自行选择可增加灵活性
- 规格不同，不一定符合用户需求


{{< callout context="danger" title="危险" icon="alert-octagon" >}}
在选购Pico核心板时，一定要注意核心板的引脚分布是否与官方一致，**否则可能会损坏显示屏模组和核心板**
{{< /callout >}}

官方版本引脚分布图如下：

{{< figure
  src="images/pico-r3-pinout.png"
  alt="（若图片过小可鼠标右键新标签页打开图片）"
  caption="（若图片过小可鼠标右键新标签页打开图片）"
>}}

{{<figure
  src="images/pico.jpg"
  caption="https://www.blendswap.com/blend/27180"
>}}

## 核心板的焊接

如果您自行准备核心板，那么在焊接排针的时候，应该以排针向下的方式焊接。

在焊接核心板排针的时候，可能会因对齐问题导致排针焊歪，此时可以使用如下3D打印模型辅助：[pico-soldering-jig.stl](http://embeddedboys.com/uploads/pico-soldering-jig.stl)
{{< figure
  src="images/pico-soldering-jig.webp"
  process="fill 480x270"
  width="160"
  sizes="75vw"
  alt=""
  caption=""
>}}

将排针放至孔中，再放上Pico，就像这样：
{{< figure
  src="images/pico-on-jig.jpg"
  process="fill 480x270"
  width="160"
  sizes="75vw"
  alt=""
  caption=""
>}}
然后就可以开始焊接啦

{{< callout context="danger" title="危险" icon="alert-octagon" >}}
不要将排针向上焊接的核心板安装到显示屏模组中！
{{< /callout >}}


## 核心板的安装与卸载

<mark>核心板USB接口方向一定要对准电路板上丝印USB标识处</mark>，将两排排针对准排母，两边同时用力按下，注意不要按坏屏幕，另一只手可以手掌托着屏幕一侧（增大受力面积），按压至最低端无法继续前进即可。 如果核心板安装后，两侧不处于同一平面，通常有一下两个原因：

1. 核心板两侧受力不均，导致核心板一边卡在排母中，此情况可以将核心板拔出重新安装。
2. 在焊接过程中，部分锡浆流入排母中造成轻微堵塞。

以上两个原因一般都不影响正常使用，如果用户在多次尝试安装后，显示模组仍存在显示问题，可以联系我们进行更换。

在移除核心板时，建议使用IC起拔器，否则核心板两端受力不均会导致核心板的排针弯曲，在将其修正前无法插入模组中。

{{< callout context="danger" title="危险" icon="alert-octagon" >}}
错误的安装可能会损坏核心板和模组！
{{< /callout >}}

## 核心板数据统计
我们测试了市面上绝大多数品牌的 Pico 核心板并进行了统计，最终发现，有些板子甚至无法满足正常开发工作，且大概率是使用了劣质的Flash芯片所造成的，通常表现为复位后无法正常启动。

在初始阶段就应该避免这些问题，这也是本文的目的。

下面我们将按照统一格式，根据推荐程度列出**经过测试**的核心板:

{{< callout context="note" title="声明" icon="alert-octagon" >}}
并非处于专业环境测试，数值仅供参考。
{{< /callout >}}

示例格式如下：

------

 （x. 名称）

（评价）

（图片）

（极限参数表格）

------

有关极限参数的疑问，请查看[性能参数/基本](/docs/performance/basic/)

### 1. 官方版本

除了接口是Micro USB，没有其他问题，如果手头上有这个的话，优先选择
{{< figure
  process="fill 480x270"
  width="160"
  sizes="75vw"
  src="images/pico-board.png"
  alt=""
>}}


| 测试项    |   数值  | 
| --- | --- |
| CPU | 单核最高420MHz 双核 416MHz |
| Flash | 105MHz / 104MHz |

### 2. 合宙 Pico 核心板

9.9包邮的时候如果买过，可以用这个，稳定性仅次于官方。
{{< figure
  src="images/luatos-pico.png"
  alt=""
>}}

| 测试项    |   数值  | 
| --- | --- |
| CPU | 单、双核 400MHz |
| Flash | 100MHz |

### YD-RP2040 核心板

这个核心板的成绩并不理想，如果不超频使用，还是可以考虑下的，毕竟板载了较多的东西。

{{< figure
  process="fill 480x270"
  src="images/yd-rp2040.webp"
  alt=""
>}}

| 测试项    |   数值  | 
| --- | --- |
| CPU | 单、双核 266MHz |
| Flash | 133MHz |


### 无名科技 Pico 核心板

稳定性跟YD-RP2040差不多，价格稍贵，颜值在线。

{{< figure
  process="fill 480x270"
  src="images/wuming-pico.webp"
  alt=""
>}}


| 测试项    |   数值  | 
| --- | --- |
| CPU | 单、双核 266MHz |
| Flash | 133MHz |

### 官方版本国产版1

9.9包邮的性价比还是不错的，虽然可能会存在跟后者相同的问题，但是稳定性优于后者。

{{< figure
  process="fill 480x270"
  src="images/pico-1.webp"
  alt=""
>}}


| 测试项    |   数值  | 
| --- | --- |
| CPU | 单、双核 400MHz |
| Flash | 100MHz |

### 官方版本国产版2

这个板子重复编程flash会发烫，然后过热期间rp2040无法正常启动，不排除是个体原因。

{{< figure
  process="fill 480x270"
  src="images/pico-2.webp"
  alt=""
>}}


| 测试项    |   数值  | 
| --- | --- |
| CPU | 单、双核 266MHz |
| Flash | 133MHz |
