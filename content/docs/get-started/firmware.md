---
title: "固件烧录"
description: ""
summary: ""
date: 2024-03-06T11:08:28Z
lastmod: 2024-03-06T11:08:28Z
draft: false
weight: 104
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

## UF2 烧录


![image](images/blink-an-led-final.gif)

图片引用自[树莓派 Pico 中文站](https://pico.org.cn/)

## DAPLink OpenOCD 烧录

1. 安装 openocd

{{< tabs "install-openocd" >}}

{{< tab "Ubuntu" >}}
```shell
sudo apt install automake autoconf build-essential texinfo libtool libftdi-dev libusb-1.0-0-
dev -y

git clone https://github.com/raspberrypi/openocd.git --recursive --branch rp2040 --depth=1

cd openocd
./bootstrap
./configure --enable-cmsis-dap
make -j12
sudo make install
```
{{< /tab >}}

{{< tab "Fedora" >}}
```shell
```
{{< /tab >}}

{{< /tabs >}}


2. 上传程序到 Pico

{{< tabs "upload-program" >}}
{{< tab "Ubuntu" >}}

```shell
sudo openocd -f interface/cmsis-dap.cfg -f target/rp2040.cfg -c "adapter speed 5000" -c "program blink.elf verify reset exit"
```

{{< /tab >}}

{{< tab "Fedora" >}}

```shell
```

{{< /tab >}}
{{< /tabs >}}

## picotool 烧录
