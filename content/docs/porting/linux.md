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

## 说明

本文用于介绍如何将此拓展板部署到Linux平台，因为此拓展板基于GPIO设计，在Linux中只目前只能使用gpiolib接口驱动，因为各平台实现方式不同，所以速度会有较大差异，效率普遍低下，但是能用😊。

相关资料仓库：[https://github.com/embeddedboys/pico_dm_qd3503728_linux](https://github.com/embeddedboys/pico_dm_qd3503728_linux)

## Luckfox Pico & Max{#luckfox_pico}

luckfox pico有多个硬件版本型号，目前我们只对最基础的版本做了适配，也就是`Luckfox Pico IPC`，我们也购买了Luckfox Pico Max，适配工作正在进行中。

luckfox pico有两个版本的SDK，分别是Buildroot和Ubuntu，我们针对这两个版本都做了适配。

### 硬件改动 飞线

在使用Luckfox Pico的时候，需要先飞两根线，**Max 版本无需飞线**。

1. 短接`GP18`和`GP22`，因为luckfox pico的GP22是NC（无连接）的，之前我短接到`GP21`，这会导致屏幕触摸时的中断信号触发屏幕复位（GP21是 FT6236的 IRQ引脚），下面的图片已经过时，不要参考。
{{<figure
  src="images/reset-wiring.png"
  process="fill 480x270"
  width="160"
  sizes="75vw"
>}}

2. 短接电阻`R3`和`R4`的左侧，使背光常开，因为luckfox pico的GP28是NC
{{<figure
  src="images/backlight-wiring.png"
  process="fill 480x270"
  width="160"
  sizes="75vw"
>}}


### Buildroot

**1. 修改设备树**

[rv1103g-luckfox-pico.dts](https://github.com/embeddedboys/pico_dm_qd3503728_linux/blob/main/luckfox-pico/rv1103g-luckfox-pico.dts)

{{< callout context="note" title="说明" icon="info-circle" >}}
Luckfox Pico、Pro、Max的引脚分布是不同的，所以不能套用同一份设备树。
{{< /callout >}}

在根节点中添加如下节点
```shell
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

修改pinctrl,设定gpio方向、驱动强度等
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
**2. 重新编译，烧录boot.img，重启**
```shell
./build.sh kernel

adb push output/image/boot.img /root
adb shell

# 此时已位于设备端，mmcblk1p4是boot分区
dd if=/root/boot.img of=/dev/mmcblk1p4 bs=1M && reboot
```
**4. 编译、加载fb驱动**
```bash
git clone https://github.com/embeddedboys/pico_dm_qd3503728_linux
cd pico_dm_qd3503728_linux

# 修改 Makefile 中这个三个变量到您设备的路径
# ARCH := arm
# CROSS_COMPILE := /home/developer/sources/luckfox-pico/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf/bin/arm-rockchip830-linux-uclibcgnueabihf-
# KERN_DIR := /home/developer/sources/luckfox-pico/sysdrv/source/kernel

make

adb push ili9488_fb.ko /root/
adb shell

# 此时已位于设备端
insmod ili9488_fb.ko
```

#### 如果您觉得太麻烦了，可以使用我们编译好的文件，但这会覆盖您设备当前的内核及设备树，操作步骤如下

1. 下载固件

[boot.img](http://embeddedboys.com/uploads/luckfox-pico/boot.img)
[ili9488_fb.ko](http://embeddedboys.com/uploads/luckfox-pico/ili9488_fb.ko)

或者到如下仓库路径下载

[https://github.com/embeddedboys/pico_dm_qd3503728_linux/tree/main/luckfox-pico](https://github.com/embeddedboys/pico_dm_qd3503728_linux/tree/main/luckfox-pico)

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
adb shell

# 此时已位于设备端
insmod /root/ili9488_fb.ko
```

5. 此时可以看到屏幕已经出现fb console

{{<figure
  src="images/luckfox-pico-buildroot-fb.jpg"
>}}

### Ubuntu

[Luckfox Pico Ubuntu server 安装桌面环境](https://www.cnblogs.com/hfwz/p/18136386)

{{<figure
  src="images/luckfox-pico-max-ubuntu.jpg"
>}}

触摸暂时还没有调通，因为我无法在该平台上使用i2c-gpio，具体原因忘记了，挺棘手的，不过还可以使用usb host连接鼠标键盘操作。

[Luckfox Pico配置为USB HOST模式](https://spotpear.cn/index/forum/detail/id/45.html#:~:text=%E6%82%A8%E5%8F%AF%E4%BB%A5%E5%B0%86%E8%AE%BE%E5%A4%87%E6%A0%91%E9%85%8D%E7%BD%AE%E4%B8%BA%20USB%20HOST%20%E6%A8%A1%E5%BC%8F%EF%BC%8C%E4%BB%A5%E4%BE%BF%E9%80%9A%E8%BF%87%20USB%20HUB%20%E6%89%A9%E5%B1%95%E5%A4%9A%E4%B8%AA%E6%8E%A5%E5%8F%A3%E3%80%82%20%E6%B8%A9%E9%A6%A8%E6%8F%90%E7%A4%BA%EF%BC%9A,%E5%8F%A3%EF%BC%8C%E4%BE%9B%E7%94%B5%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E5%BE%AE%E9%9B%AAPico%20To%20HAT%20%E4%B8%8A%E7%9A%84Micro%20usb%20%E4%BE%9B%E7%94%B5%E6%88%96%E8%80%85%E6%98%AF%E9%80%9A%E8%BF%87%20GPIO%20%E4%BE%9B%E7%94%B5%EF%BC%8C%E4%BE%9B%E7%94%B5%E9%9C%80%E8%B0%A8%E6%85%8E%E4%BB%A5%E5%85%8D%E6%8D%9F%E5%9D%8F%E5%BC%80%E5%8F%91%E6%9D%BF)

但是这样就没法通过adb调试了，所以你需要事先写一个systemd服务，在开机初始化好一切需要的事务。 Max 或者 Pro版本因为有网口，所以还可以通过网络登陆设备。

## Milk-V Duo{#milk-v-duo}

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

待添加，我们这里没有此开发板，但他们都使用同一份sdk，所以应该是大同小异的，用户可以先自行尝试移植。
