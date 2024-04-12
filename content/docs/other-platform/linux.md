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

本文用于介绍如何将此拓展板部署到Linux平台，因为此拓展板基于GPIO设计，在Linux中只目前只能使用gpiolib接口驱动，因为各平台实现方式不同，所以速度会有较大差异，效率普遍低下，但是能用😊。

相关资料仓库：[https://github.com/embeddedboys/pico_dm_qd3503728_linux](https://github.com/embeddedboys/pico_dm_qd3503728_linux)

## Luckfox Pico{#luckfox_pico}

luckfox pico有多个硬件版本型号，目前我们只对最基础的版本做了适配，也就是`Luckfox Pico IPC`，我们也购买了Luckfox Pico Max，适配工作正在进行中。

luckfox pico有两个版本的SDK，分别是Buildroot和Ubuntu，我们针对这两个版本都做了适配。

### 硬件改动 飞线

在使用Luckfox Pico的时候，需要先飞两根线

1. 短接`GP21`和`GP22`，因为luckfox pico的GP22是NC（无连接）的
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

1. 修改设备树

在根节点中添加如下节点
```shell
ili9488 {
  status = "okay";
  compatible = "ultrachip,uc8253";
  pinctrl-names = "default";
  pinctrl-0 = <&i80_pins>;
  fps = <30>;
  buswidth = <8>;
  debug = <0x7>;
  db =  <&gpio1 RK_PB2 GPIO_ACTIVE_HIGH>,
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
        reset = <&gpio4 RK_PA4 GPIO_ACTIVE_HIGH>;    //RES
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
				<4 RK_PA4 RK_FUNC_GPIO &pcfg_pull_none_drv_level_0>;
		};
	};
};

```

2. 修改内核配置，开启fb console支持
3. 重新编译，烧录boot.img，重启
```shell
./build.sh kernel
```
4. 编译，加载fb驱动

#### 如果您觉得太麻烦了，可以使用我们编译好的文件，但这会覆盖您设备当前的内核及设备树，操作步骤如下

1. 下载固件

[boot.img](http://embeddedboys.com/uploads/luckfox-pico/boot.img)

[ili9488_fb.ko](http://embeddedboys.com/uploads/luckfox-pico/ili9488_fb.ko)

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

{{<figure
  src="images/luckfox-pico-ubuntu.jpg"
>}}

## Milk-V Duo{#milk-v-duo}

### 硬件改动 飞线

### Buildroot