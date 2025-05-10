---
title: "Linux"
description: ""
summary: ""
date: 2024-03-27T22:46:04Z
lastmod: 2024-03-27T22:46:04Z
draft: false
weight: 1102
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

本文用于介绍如何将此拓展板移植到Linux平台，我们提供了两种方式来驱动此拓展板：

- 使用 GPIO 直接驱动

- 通过 USB 接口驱动

本文篇幅较长，推荐读者使用目录快速跳转到感兴趣的部分

# 使用 GPIO 直接驱动

因为此拓展板基于Raspberry Pico + I8080 16bit接口设计，所以市面上与 Raspberry Pico 引脚兼容的开发板，理论上都可以驱动。

为了保持兼容性，在Linux中只使用GPIO进行驱动。 因为各平台的gpio驱动实现方式不同，所以IO翻转速度可能会有较大差异，效率普遍低下，但是能用😊。

我们提供了两种 linux 驱动，分别为 Framebuffer 和 DRM 驱动，这里有一个表格可以让你快速了解这两种驱动之间的差异。

| 特性 | Framebuffer (fbdev) | DRM (Direct Rendering Manager) |
| --- | --- | --- |
| 开发初衷 | 提供简单的图形输出接口 | 管理现代 GPU 并支持硬件加速 |
| 图形加速支持 | 通常不支持，仅支持软件渲染 | 完整支持硬件加速（通过 GPU）|
| 渲染架构 | 没有明确的 GPU 渲染流程，应用直接写 framebuffer | 支持 render nodes，结合 Mesa、Wayland、X11 等使用 |
| 使用场景 | 嵌入式、资源受限环境 | 桌面系统、图形丰富的应用场景 |
| 未来趋势 | 被逐步淘汰 | 发展方向，推荐使用 |

本项目推荐使用 DRM 驱动，至于具体原因，感兴趣的同学可以参考本文档闲聊中的[（待更新）FrameBuffer驱动对于低分辨率屏幕的局限性]()

相关源码仓库：[https://github.com/embeddedboys/pico_dm_qd3503728_linux](https://github.com/embeddedboys/pico_dm_qd3503728_linux)

## Luckfox Pico {#luckfox_pico}

luckfox pico有两个版本的SDK，分别是Buildroot和Ubuntu，我们针对这两个版本都做了适配。

### 硬件改动 飞线

在使用Luckfox Pico的时候，需要先飞两根线

1. 短接`GP18`和`GP22`，因为luckfox pico的GP22是NC（无连接）的

2. 短接电阻`R3`和`R4`的左侧，使背光常开，因为luckfox pico的GP28是NC
{{<figure
  src="images/backlight-wiring.png"
  process="fill 480x270"
  width="160"
  sizes="75vw"
>}}

### Buildroot

**1. 修改设备树**

你可以直接下载下面的设备树文件，替换到luckfox pico sdk中，路径是：`sysdrv/source/kernel/arch/arm/boot/dts/rv1103g-luckfox-pico.dts`

[rv1103g-luckfox-pico.dts](https://github.com/embeddedboys/pico_dm_qd3503728_linux/blob/main/luckfox-pico/rv1103g-luckfox-pico.dts)

{{< callout context="note" title="说明" icon="info-circle" >}}
Luckfox Pico、Pro、Max的引脚分布是不同的，所以不能套用同一份设备树。
{{< /callout >}}

1.1 在根节点中添加如下节点
```shell
	chosen {
		bootargs = "earlycon=uart8250,mmio32,0xff4c0000 console=ttyFIQ0 console=tty0 root=/dev/mmcblk1p7 rootwait snd_soc_core.prealloc_buffer_size_kbytes=16 coherent_pool=0";
    };

	ili9488 {
		status = "okay";
		compatible = "ilitek,ili9488";
		pinctrl-names = "default";
		pinctrl-0 = <&i80_pins>;
		fps = <30>;
		buswidth = <8>;
		debug = <0x7>;
		db =	<&gpio1 RK_PB2 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PB3 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC7 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC6 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC5 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC4 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PD2 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PD3 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PA2 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC0 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC1 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC2 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PC3 GPIO_ACTIVE_HIGH>,
				<&gpio0 RK_PA4 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PD0 GPIO_ACTIVE_HIGH>,
				<&gpio1 RK_PD1 GPIO_ACTIVE_HIGH>;

				dc = <&gpio4 RK_PA3 GPIO_ACTIVE_HIGH>;      //RS
				wr = <&gpio4 RK_PA2 GPIO_ACTIVE_HIGH>;
				reset = <&gpio4 RK_PA6 GPIO_ACTIVE_HIGH>;    //RESET
	};
```

bootargs 添加 `console=tty0` 参数，使内核日志和控制台打印到屏幕上。

1.2. 修改pinctrl,设定gpio方向、驱动强度等
```shell
&pinctrl {
	i80 {
		i80_pins: i80-pins {
			rockchip,pins =
				<1 RK_PB2 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PB3 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC7 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC6 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC5 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC4 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PD2 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PD3 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PA2 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC0 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC1 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC2 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PC3 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<0 RK_PA4 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PD0 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<1 RK_PD1 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<4 RK_PA3 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<4 RK_PA2 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>,
				<4 RK_PA6 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>;
		};
	};
};
```

1.3. 删除根节点中冲突的gpio节点，如果不这么做，会导致驱动无法申请到冲突的GPIO。
你可以使用diff工具查看你修改后的设备树与我们提供的设备树之间的差异

**2. 重新编译，烧录boot.img，重启**
```shell
./build.sh kernel

adb push output/image/boot.img /root
adb shell

# 此时已位于设备端，mmcblk1p4是使用SD卡启动时的boot分区
dd if=/root/boot.img of=/dev/mmcblk1p4 bs=1M && reboot
```

<!-- 在这里添加带SPI NAND 版本的pico 烧录时 boot 分区的路径 -->

**4. 编译、加载fb驱动**
```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_linux
cd pico_dm_qd3503728_linux

# 修改 Makefile 中这个三个变量到您设备中对应的路径
# ARCH := arm
# CROSS_COMPILE := /home/developer/sources/luckfox-pico/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf/bin/arm-rockchip830-linux-uclibcgnueabihf-
# KERN_DIR := /home/developer/sources/luckfox-pico/sysdrv/source/kernel

make -f Makefile.luckfox-pico

adb push ili9488_fb.ko /root/
adb shell

# 此时已位于设备端
insmod ili9488_fb.ko
```

**5. （可选）配置系统启动时自动加载驱动**
```shell

```

#### 如果您觉得太麻烦了，可以使用我们编译好的文件，但这会覆盖您设备当前的内核及设备树，操作步骤如下

1. 下载固件

[boot.img](https://github.com/embeddedboys/pico_dm_qd3503728_linux/blob/main/luckfox-pico/boot.img)
[ili9488_fb.ko](https://github.com/embeddedboys/pico_dm_qd3503728_linux/blob/main/luckfox-pico/ili9488_fb.ko)


2. 将固件推送至设备中
```shell
adb push boot.img /root/
adb push ili9488_fb.ko /root/
```

3. 使用adb访问设备终端，烧录boot.img
```shell
adb shell

# 此时已位于设备端
dd if=/root/boot.img of=/dev/mmcblk1p4 bs=1M && reboot
```

4. 待设备重启完成后，再次使用adb访问设备终端，加载设备驱动
```shell
adb wait-for-device && adb shell

# 此时已位于设备端
insmod /root/ili9488_fb.ko
```

5. 此时可以看到屏幕已经出现fb console

{{<figure
  src="images/luckfox-pico-buildroot-fb.jpg"
>}}

### Ubuntu

设备树根 Buildroot 章节中保持一致即可，如果你不使用摄像头，可以将 `RK_BOOTARGS_CMA_SIZE` 大小修改为1M，参考[这里](https://wiki.luckfox.com/zh/Luckfox-Pico/Luckfox-Pico-RV1103/Luckfox-Pico-Plus-Mini/Luckfox-Pico-SDK#13-sdk%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

官方SDK编译出的Ubuntu镜像默认是无桌面环境的，你可以参考下面的文章来了解如何在Ubuntu Server上手动安装桌面环境。
[Luckfox Pico Ubuntu server 安装桌面环境](https://www.cnblogs.com/hfwz/p/18136386)

{{<figure
  src="images/luckfox-pico-max-ubuntu.jpg"
>}}

触摸暂时还没有调通，因为我无法在luckfox pico的`GPIO4_C1`,`GPIO4_C0`这两个引脚上正常使用i2c-gpio。 这两个引脚的电压范围为0 ~ 1.8V，并且被用于ADC功能。

你可以参考如下链接中的内容将luckfox pico的usb设置为主机模式，这样你就可以连接鼠标键盘等USB设备了。

[Luckfox Pico配置为USB HOST模式](https://spotpear.cn/index/forum/detail/id/45.html#:~:text=%E6%82%A8%E5%8F%AF%E4%BB%A5%E5%B0%86%E8%AE%BE%E5%A4%87%E6%A0%91%E9%85%8D%E7%BD%AE%E4%B8%BA%20USB%20HOST%20%E6%A8%A1%E5%BC%8F%EF%BC%8C%E4%BB%A5%E4%BE%BF%E9%80%9A%E8%BF%87%20USB%20HUB%20%E6%89%A9%E5%B1%95%E5%A4%9A%E4%B8%AA%E6%8E%A5%E5%8F%A3%E3%80%82%20%E6%B8%A9%E9%A6%A8%E6%8F%90%E7%A4%BA%EF%BC%9A,%E5%8F%A3%EF%BC%8C%E4%BE%9B%E7%94%B5%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E5%BE%AE%E9%9B%AAPico%20To%20HAT%20%E4%B8%8A%E7%9A%84Micro%20usb%20%E4%BE%9B%E7%94%B5%E6%88%96%E8%80%85%E6%98%AF%E9%80%9A%E8%BF%87%20GPIO%20%E4%BE%9B%E7%94%B5%EF%BC%8C%E4%BE%9B%E7%94%B5%E9%9C%80%E8%B0%A8%E6%85%8E%E4%BB%A5%E5%85%8D%E6%8D%9F%E5%9D%8F%E5%BC%80%E5%8F%91%E6%9D%BF)

但是这样就没法通过adb调试了，所以你需要事先写一个systemd服务，在开机初始化好一切需要的事务。 Luckfox Max 或者 Pro版本因为有网口，所以还可以通过网络登陆设备。

## （待更新）Luckfox Pico Max

这里只展示与 pico 差异的地方

### Buildroot

**1. 修改设备树**

1.1 在根节点中添加如下节点
```c
```

1.2. 修改pinctrl,设定gpio方向、驱动强度等
```c
```

## （待更新）Luckfox Lyra & Plus

Luckfox Lyra 主控采用Rockchip RK3506 处理器，该处理器采用 22nm 制程工艺，搭载了4 核 32 位 CPU（包括 3×Cortex-A7 和 1×Cortex-M0），丰富的接口扩展，适用于多种应用领域，包括物联网设备、智能音频、智能显示、工业控制和教育设备等。Luckfox Lyra 支持 Buildroot 和 Ubuntu22.04 系统。

### 开发环境搭建

请先参考如下链接中的内容搭建好开发环境:
[https://wiki.luckfox.com/zh/Luckfox-Lyra/SDK](https://wiki.luckfox.com/zh/Luckfox-Lyra/SDK)

官方提供的 SDK 下载链接：[https://pan.baidu.com/s/1KyPieuwh63ynd-O96ChlDA?pwd=mzqk](https://pan.baidu.com/s/1KyPieuwh63ynd-O96ChlDA?pwd=mzqk)

我们开发者使用的系统是 Ubuntu 24.04.2 LTS

解压 Luckfox Lyra SDK 到指定目录
```bash
mkdir -p ~/luckfox/lyra
tar xvf ~/Downloads/Luckfox_Lyra_SDK_250429.tar.gz -C lyra
```

拉取工程
```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_linux.git ~/pico_dm_qd3503728_linux
```

修改 SDK 中的内核工程

```bash
cd ~/luckfox/lyra

cp ~/pico_dm_qd3503728_linux/luckfox-lyrark3506g-luckfox-lyra-sd.dts ~/luckfox/lyra/kernel/arch/arm/boot/dts/rk3506g-luckfox-lyra-sd.dts
```


编译驱动 ko 文件
```bash
cd ~/pico_dm_qd3503728_linux/luckfox-lyra
make
```

## （过时的）Milk-V Duo{#milk-v-duo}

[https://github.com/embeddedboys/pico_dm_qd3503728_linux](https://github.com/embeddedboys/pico_dm_qd3503728_linux)

### 硬件改动 飞线
飞线跟luckfox pico章节中保持一致即可

### Buildroot

**1. 设置好buildroot工程**
```bash
sudo apt install -y pkg-config build-essential ninja-build automake autoconf libtool wget curl git gcc libssl-dev bc slib squashfs-tools android-sdk-libsparse-utils jq python3-distutils scons parallel tree python3-dev python3-pip device-tree-compiler ssh cpio fakeroot libncurses5 flex bison libncurses5-dev genext2fs rsync unzip dosfstools mtools tcl openssh-client cmake expect
```

```bash
git clone https://github.com/milkv-duo/duo-buildroot-sdk.git --depth=1
```

至少运行`build.sh`一次.
```bash
./build.sh lunch

# 选1. milkv-duo-sd
# 之后会开始整个编译流程
```

**2. 加载环境变量**
```bash
source device/milkv-duo-sd/boardconfig.sh

source build/milkvsetup.sh
defconfig cv1800b_milkv_duo_sd
```

**3. 编译内核、设备树**

```bash
# 拷贝我们提供的设备树文件到内核工程中
cp /home/developer/embeddedboys/pico_dm_qd3503728_linux/milk-v-duo/cv1800b_milkv_duo_sd.dts \
    linux_5.10/arch/riscv/boot/dts/cvitek/

cd ~/sources/duo-buildroot-sdk/linux_5.10/build/kernel_output
make dtbs
```

**4. 重新烧录内核、设备树**
```bash
# 注意替换成你们当前环境的目录
cd ~/sources/duo-buildroot-sdk/ramdisk/build/cv1800b_milkv_duo_sd/workspace
cp ~/sources/duo-buildroot-sdk/linux_5.10/build/kernel_output/arch/riscv/boot/dts/cvitek/cv1800b_milkv_duo_sd.dtb .
cp ~/sources/duo-buildroot-sdk/linux_5.10/build/kernel_output/arch/riscv/boot/Image .

rm -rf Image.lzma
lzma -k Image
mkimage -f multi.its boot.sd

scp boot.sd root@192.168.42.1:/boot/
```

然后重启duo板子，应用新的内核。

**5. 编译加载拓展板驱动**
```bash
cd pico_dm_qd3503728_linux
make -f Makefile.milk-v-duo

scp milk-v-duo/test.sh root@192.168.42.1:~/
scp ili9488_fb.ko root@192.168.42.1:~/

# at device side start
./test.sh
# at device side end
```

脚本`test.sh`的内容
```bash
#!/bin/sh

duo-pinmux -w GP0/GP0 > /dev/null 2>&1
duo-pinmux -w GP1/GP1 > /dev/null 2>&1
duo-pinmux -w GP2/GP2 > /dev/null 2>&1
duo-pinmux -w GP3/GP3 > /dev/null 2>&1
duo-pinmux -w GP4/GP4 > /dev/null 2>&1
duo-pinmux -w GP5/GP5 > /dev/null 2>&1
duo-pinmux -w GP6/GP6 > /dev/null 2>&1
duo-pinmux -w GP7/GP7 > /dev/null 2>&1
duo-pinmux -w GP8/GP8 > /dev/null 2>&1
duo-pinmux -w GP9/GP9 > /dev/null 2>&1
duo-pinmux -w GP10/GP10 > /dev/null 2>&1
duo-pinmux -w GP11/GP11 > /dev/null 2>&1
duo-pinmux -w GP12/GP12 > /dev/null 2>&1
duo-pinmux -w GP13/GP13 > /dev/null 2>&1
duo-pinmux -w GP14/GP14 > /dev/null 2>&1
duo-pinmux -w GP15/GP15 > /dev/null 2>&1

rmmod ili9488_fb.ko
insmod ili9488_fb.ko
```

milk-v duo 使用一个名为`duo-pinmux`的用户空间工具来进行引脚复用，非常的奇怪。
我记得是当时没法从dts中设置pinctrl引脚复用来着，然后翻看他们文档说用上面这个工具。

**6. 编译并运行lvgl demo**

由于buildroot的工具链指向特定的动态库，需要在板子中进行软链接

（最新的sdk已经修复这个问题）
```bash
# at device side start
cd /lib
ln -sf ../usr/lib64v0p7_xthead/lp64d/libc.so ./ld-musl-r
iscv64xthead.so.1
# at device side end
```

```bash
git clone https://github.com/lvgl/lv_port_linux_frame_buffer.git

cd lv_port_linux_frame_buffer
git checkout release/v8.2

# 这一步可能耗时很长，因为需要从github拉取lvgl、lv_drivers子模块
git submodule update --init

# 修改 main.c
# disp_drv.hor_res    = 480;
# disp_drv.ver_res    = 320;

# 修改 lv_conf.h
# #define LV_COLOR_DEPTH 16

make CC=/home/developer/sources/duo-buildroot-sdk/host-tools/gcc/riscv64-linux-musl-x86_64/bin/riscv64-unknown-linux-musl-gcc -j20

scp demo root@192.168.42.1:~/

# at device side start
./demo
# at device side end
```

此时lvgl demo应该已经在拓展板上显示出来了。

**7. 跑分、 性能优化**

TODO

ili9488设备树节点
```c
	ili9488 {
        status = "okay";
        compatible = "ilitek,ili9488";
		// pinctrl-names = "default";
		// pinctrl-0 = <&i80_pins>;
        fps = <30>;
        buswidth = <16>;
        // debug = <0x7>;
        db =  <&porta 28 GPIO_ACTIVE_HIGH>,
        	  <&porta 29 GPIO_ACTIVE_HIGH>,
        	  <&porte 26 GPIO_ACTIVE_HIGH>,
        	  <&porte 25 GPIO_ACTIVE_HIGH>,
        	  <&porte 19 GPIO_ACTIVE_HIGH>,
        	  <&porte 20 GPIO_ACTIVE_HIGH>,
        	  <&porte 23 GPIO_ACTIVE_HIGH>,
        	  <&porte 22 GPIO_ACTIVE_HIGH>,
        	  <&porte 21 GPIO_ACTIVE_HIGH>,
        	  <&porte 18 GPIO_ACTIVE_HIGH>,
        	  <&portc 9 GPIO_ACTIVE_HIGH>,
        	  <&portc 10 GPIO_ACTIVE_HIGH>,
        	  <&porta 16 GPIO_ACTIVE_HIGH>,
        	  <&porta 17 GPIO_ACTIVE_HIGH>,
        	  <&porta 14 GPIO_ACTIVE_HIGH>,
        	  <&porta 15 GPIO_ACTIVE_HIGH>;

        dc = <&porta 27 GPIO_ACTIVE_HIGH>;      //RS
        wr = <&porta 25 GPIO_ACTIVE_HIGH>;
        reset = <&porte 4 GPIO_ACTIVE_HIGH>;    //RES
	};
```

## Milk-V Duo 256M{#milk-v-duo-256M}

待添加。虽然我们目前没有此开发板，但它们都使用相同的SDK，所以应该差别不大。用户可以先自行尝试移植。

# 通过 USB 接口驱动 （USB Display）

# 驱动分析

## FrameBuffer

## DRM
