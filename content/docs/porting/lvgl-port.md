---
title: "LVGL移植"
description: ""
summary: ""
date: 2024-03-23T01:02:33Z
lastmod: 2024-03-23T01:02:33Z
draft: false
weight: 1104
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 写在前面

如果您对LVGL十分感兴趣，我非常建议自己尝试一下移植LVGL，本文将以通俗易懂的方式，向大家介绍LVGL移植的过程，希望读者能在本文的辅助下，独自完成LVGL的移植。在本文的最后部分会向大家介绍有关性能优化的细节。

事实上，LVGL的每个大版本，driver部分的接口都有较大的变化，所以我们选择了两个目前最新的release版本讲述移植过程，分别是 `v8.3` 和 `v9`。

## 准备工作

如果你想要在你的显示模组上移植LVGL，最先应该做的是将其点亮。 比如实现一个描点函数。

为了验证屏幕处于可用状态，我们通常会在HAL层实现一些基本功能，比如复位、初始化、通常分为以下几步：

1. 复位显示模组
2. 向显示模组写入初始化序列
3. 设置绘制区域
4. 发送颜色数据

## LVGL 8.3

## LVGL 9

## 性能优化