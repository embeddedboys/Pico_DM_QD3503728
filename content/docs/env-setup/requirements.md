---
title: "安装依赖"
description: ""
summary: ""
date: 2024-03-09T05:09:17Z
lastmod: 2024-03-09T05:09:17Z
draft: false
weight: 202
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## 部署 pico-sdk

### 1. 安装前置依赖

{{< tabs "install-sdk-requirements" >}}
{{< tab "Ubuntu" >}}

```bash {title="安装依赖"}
sudo apt install cmake gcc-arm-none-eabi libnewlib-arm-none-eabi libstdc++-arm-none-eabi-newlib
```

{{< /tab >}}

{{< tab "Fedora" >}}

```shell
```

{{< /tab >}}
{{< /tabs >}}

### 2. 下载 pico-sdk 源码

{{< details "国内用户" >}}
```shell
git clone https://gitee.com/embeddedboys/pico-sdk $HOME/pico-sdk
```
{{< /details >}}


{{< details "国外用户" >}}
```shell
git clone https://github.com/raspberrypi/pico-sdk/ $HOME/pico-sdk
```
{{< /details >}}

-------------


### 3. 拉取 pico-sdk 子仓库代码

```shell
cd ~/pico-sdk
git submodule update --init
```

### 4. 配置环境变量 {#pico-sdk-env}

对于bash

```shell
echo "export PICO_SDK_PATH=$HOME/pico-sdk" >> ~/.bashrc
source ~/.bashrc
```

对于zsh

```shell
echo "export PICO_SDK_PATH=$HOME/pico-sdk" >> ~/.zshrc
source ~/.zshrc
```

{{< callout context="note" title="说明" icon="info-circle" >}} 假设当前使用的是 bash {{< /callout >}}