---
title: "LVGL BenchMark"
description: ""
summary: ""
date: 2024-03-09T08:01:06Z
lastmod: 2024-03-09T08:01:06Z
draft: false
weight: 999
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

| 测试环境 | |
| --- | --- |
| CPU | 400MHz |
| Flash | 100 MHz |
| 电压 | 1.25V |
| 分辨率 | 480x320 |
| BPP | 16 |
| Buffer 大小 | 1/4 屏幕 |
| Buffer 数量 | 2 |

- Before: 裸机版本
- After : FreeRTOS 版本

{{< figure src="images/benchmark_result.png" alt="" >}}

- X: 测试项目

- Y: FPS (Frame per second) （越高越好）

{{< callout context="note" title="说明" icon="info-circle" >}}
裸机版本的 Buffer大小为1/2屏幕  Buffer数量为1
{{< /callout >}}
