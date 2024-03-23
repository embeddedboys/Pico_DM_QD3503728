---
title: "安装工具（可选）"
description: ""
summary: ""
date: 2024-03-09T06:28:44Z
lastmod: 2024-03-09T06:28:44Z
draft: false
weight: 203
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

正所谓工欲善其事必先利其器，本文将介绍一些主流的开发工具。

如果您已有习惯使用的工具，可跳过本章节。

## 通用

### Oh My Zsh

强大无比的终端

1. 安装本体

```shell
sudo apt install zsh -y

# 通过curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 通过wget
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

2. 安装主题

```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

中国用户可以使用 gitee.com 上的官方镜像加速下载.

```shell
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

3. 配置主题

执行完上一步之后，输入如下指令，会弹出配置窗口，根据提示选择即可。

```shell
source ~/.zshrc
```

简单配置完成后效果如下：

{{< figure src="images/powerlevel10k.png" alt="" >}}

4. 安装插件

zsh-autosuggestions - zsh 的 Fish-like 的自动建议

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions
echo "source ${(q-)PWD}/zsh-autosuggestions/zsh-autosuggestions.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
```

{{< figure src="images/zsh-autosuggestions.png" alt="" >}}

--------

zsh-syntax-highlighting - zsh 的 Fish-like 的语法高亮

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
echo "source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
```

{{< figure src="images/zsh-syntax.png" alt="" >}}

5. 使配置生效

```shell
source ~/.zshrc
```

### fzf

命令行模糊查找器

```shell
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

选项默认选y即可

使用方法：`CTRL + R` 唤出历史命令记录，此时键入字符可进行过滤。

更多用法参考[usage](https://github.com/junegunn/fzf?tab=readme-ov-file#usage)

{{< figure src="images/fzf.png" alt="" >}}
{{< figure src="images/fzf2.png" alt="" >}}

## VScode相关

### clangd（未完）

本项目的所有cmake工程都开启了`compile_commands.json`支持，可以使用clangd来进行代码索引

1. 打开VScode，安装clangd插件

2. 安装clangd服务器

3. 手动触发clangd生成代码索引

4. 测试功能

## Pico相关

### picotool

树莓派官方开发的用于pico的工具集，可用于查看binary info，load/dump二进制等。

```shell
git clone https://github.com/raspberrypi/picotool
```

用法示例：

```shell
$ picotool load blink.uf2
Loading into Flash: [==============================]  100%
```
{{< callout context="note" title="说明" icon="info-circle" >}}
此方法同样需要RP2040处于下载模式
{{< /callout >}}