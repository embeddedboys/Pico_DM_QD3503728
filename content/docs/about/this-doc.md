---
title: "本文档"
description: ""
summary: ""
date: 2024-04-01T07:22:37+08:00
lastmod: 2024-04-01T07:22:37+08:00
draft: false
weight: 1202
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

本文档优先更新至Gitub，通过触发Action完成部署：

[https://embeddedboys.github.io/Pico_DM_QD3503728](https://embeddedboys.github.io/Pico_DM_QD3503728)

我们的服务器会每30分钟检查一次文档更新，同步至如下链接，供国内用户访问:

[http://embeddedboys.com/Pico_DM_QD3503728](http://embeddedboys.com/Pico_DM_QD3503728)

## 构建本文档

安装[nvm](https://github.com/nvm-sh/nvm)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

重新加载环境变量
```bash
source ~/.bashrc
# 或者
source ~/.zshrc
```

通过nvm安装node
```bash
nvm install node
```

克隆编译构建文档
```bash
git clone https://github.com/embeddedboys/Pico_DM_QD3503728
cd Pico_DM_QD3503728

npm install
rm -rf public && npm run build
```

或者在本地启动服务器
```bash
rm -rf public && npm run dev
```
访问[localhost:1313](localhost:1313)
