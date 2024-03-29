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

图解步骤：

1. 准备好要烧录的UF2文件
2. 按住核心板上的BOOTSEL按键，插入USB线缆
3. 电脑识别到名为RPI-RP2的可移动存储设备
4. 将要烧录的文件拖放至名为RPI-RP2的可移动存储设备中
5. 等待传输完成，程序自动执行

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

note：
大部分情况下，该方式速度并不会快于UF2方式，尤其是在您使用的DAPLINK传输速度较慢的情况下。

## JLink OpenOCD烧录

（待添加）

PS：我的jlink正在用于其他项目，因为插了一大堆杜邦线，所以暂时不想拆掉（懒），但是我测试过，jlink的烧写速度应该是最快的了。 :joy:

## picotool 烧录

（待添加）

此方式需要RP2040处于BOOTSEL模式