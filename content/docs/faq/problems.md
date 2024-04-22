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

在排除程序问题的情况下，例如空指针等，可能是超频过度导致的，可通过降低超频幅度解决，您也可以修改工程中的超频参数。

我们不知道您使用的Pico核心板上的物料信息，例如，有的Flash芯片无法达到与官方版本标定的速度。 但是也不必担心，一般来说，这部分差距不会影响使用体验。

我们也不能排除MCU或Flash损坏的可能，如果按住BOOTSEL键之后上电，电脑能检测到RPI-RP2设备，说明MCU没有问题。

推荐使用合宙家的Pico开发板，这也是我们带核心板版本的默认配置，经测试可以稳定使用。

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

### 我在使用git拉取github工程时，出现如下错误

```bash
```

这通常是由于您的代理无法通过22端口访问github导致的，修改`~/.ssh/config`文件，加入如下内容
```bash
Host github.com
    Hostname ssh.github.com
    Port 443
    User git
```

你可以通过再次连接到 GitHub.com 来测试这是否有效：

```
$ ssh -T git@github.com
> Hi USERNAME! You've successfully authenticated, but GitHub does not
> provide shell access.
```

参考连接：[在 HTTPS 端口使用 SSH](https://docs.github.com/zh/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)