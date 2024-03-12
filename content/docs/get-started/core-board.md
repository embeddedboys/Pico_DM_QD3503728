---
title: "核心板的选择（必读）"
description: ""
summary: ""
date: 2024-03-12T10:10:16Z
lastmod: 2024-03-12T10:10:16Z
draft: false
weight: 821
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 说明

我们默认不提供核心板，主要有如下几个原因：

- 用户自行选择可增加灵活性
- 核心板利润过低，市场均价10元左右
- 规格不同，不一定符合用户需求


{{< callout context="danger" title="Danger" icon="alert-octagon" >}}
在选购Pico核心板时，一定要注意核心板的引脚分布是否与官方一致，**否则可能会损坏显示屏模组和核心板**
{{< /callout >}}

官方版本引脚分布图如下：

{{< figure
  src="images/pico-r3-pinout.png"
  alt="（若图片过小可鼠标右键新标签页打开图片）"
  caption="（若图片过小可鼠标右键新标签页打开图片）"  
>}}

我们测试了市面上绝大多数品牌的 Pico 核心板并进行了统计，最终发现，有些核心板的稳定性是有待商榷的。

##  统计

下面我们将统一格式，按照推荐程度列出**经过测试**的核心板:

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

| 测试项    |   数值  | 
| --- | --- |
| CPU | 单、双核 400MHz |
| Flash | 100MHz |