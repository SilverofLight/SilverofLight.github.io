---
layout: post
title: conda 管理 python 环境
date: 2025-08-06
author: SL
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
  - python
  - conda
  - 虚拟环境
  - linux
---

# 前言

在使用 python 编写不同项目时，可能会使用不同的 python 版本，或者使用相互冲突的 python 包，此时我们会使用到 python 的虚拟环境。

Anaconda 包含一个名为 conda 的包管理器，可以用于安装、更新和管理软件包，使用 conda 可以轻松地创建多个独立的 python 环境，然后可以在不同环境之间自由切换，避免版本冲突。

# 安装

在 Arch linux 中，直接从 aur 安装：

```bash
yay -S python-conda
```

# 配置

使用 `conda config` 来配置 conda, 例如：

```bash
conda config --add channels defaults
conda config --set solver classic
```

# 使用

## 创建

```bash
conda create --name env1
```

使用这条命令即可以创建一个名为 env1 的虚拟环境，默认和系统原 python 版本一致。

如果需要一个特定版本的环境：

```bash
conda create --name env1 python=3.8
```

这样可以创建一个 python 版本为 3.8 的环境。

## 进出虚拟环境

进入环境命令：

```bash
conda activate env1
```

退出环境：

```bash
conda deactivate
```

## 安装 pip

进入虚拟环境之后，使用命令：

```bash
conda install pip
```

即可安装 pip。

同理也可以使用 `conda install` 安装其他包。

## 其他命令

列出所有环境：

```bash
conda env list
```

删除一个环境：

```bash
conda remove --name env1 --all
```
