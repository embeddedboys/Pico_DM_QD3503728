---
title: "Basic"
description: ""
summary: ""
date: 2024-03-09T08:11:13Z
lastmod: 2024-03-09T08:11:13Z
draft: false
weight: 800
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

数据来源自 [https://youtu.be/G2BuoFNLoDM?si=toe516Kl7rsoO3Ij](https://youtu.be/G2BuoFNLoDM?si=toe516Kl7rsoO3Ij)

## 电流与时钟速度关系

{{< figure src="images/current_vs_clk_speed.png" alt="" >}}

## 指定电压下的最大时钟速度

{{< figure src="images/max_clk_speed_vs_voltage.png" alt="" >}}

{{< callout context="caution" title="注意" icon="alert-triangle" >}}
在设置CPU频率时，还应注意是否符合Flash的工作频率，公式如下：

Flash Clock = SYS_CLK / PICO_FLASH_SPI_CLKDIV

例如：Fclk = 125MHz / 2 = 62.5MHz

该宏 PICO_FLASH_SPI_CLKDIV 默认值为2

您完全不需要担心，因为我们的工程会根据超频预设自动调整其大小
{{< /callout >}}