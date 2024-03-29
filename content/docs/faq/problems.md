---
title: "常见问题"
description: ""
summary: ""
date: 2024-03-21T21:20:06Z
lastmod: 2024-03-21T21:20:06Z
draft: false
weight: 601
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

### 为什么我的 Pico 无法启动？

在排除程序问题的情况下，例如空指针等，可能是超频过度导致的，可通过降低超频幅度解决。 也不排除MCU或Flash损坏的可能，如果按住BOOTSEL键之后上电，电脑能检测到设备，说明MCU没有问题。

### 我在编译工程时，出现如下错误

```shell
CMake Error at pico_sdk_import.cmake:46 (message):
  PICO SDK location was not specified.  Please set PICO_SDK_PATH or set
  PICO_SDK_FETCH_FROM_GIT to on to fetch from git.
Call Stack (most recent call first):
  CMakeLists.txt:12 (include)


-- Configuring incomplete, errors occurred!
```

这通常是由于没有正确设置`PICO_SDK_PATH`环境变量导致的，可参考[这里](/docs/env-setup/安装依赖/#pico-sdk-env)解决。