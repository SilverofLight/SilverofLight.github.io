---
layout: post
title: Python + wofi 实现本地加密的书签管理
date: 2025-08-09
author: SL
header-img: img/home-bg.jpg
catalog: true
tags:
  - python
  - wofi
  - wayland
  - linux
  - 书签
---

# 前言

我已经使用 wofi 管理我的网页书签很久了，一直以来都是使用 bash 脚本的形式，把书签作为一个表格明文写在脚本中，和我的配置文件一起托管在 github ，这样每次我要新加一个书签或者删除书签，都需要打开这个脚本进行修改，很麻烦，并且随着我保存的比较私密的网站越来越多，这样的明文托管让我感觉心理不舒服，于是决定写一个可以加密的书签脚本。

首先想到的就是使用 python 和 sqlite ，使用 python 的 sqlcipher3 可以用来加密 sqlite 数据库，可以很方便得实现我需要的功能。

# 思路

首先在一个 python 虚拟环境中下载 sqlcipher3 ，之后我创建了两个 python 文件：`~/.config/hypr/scripts/bookmark.py` 用于配合 wofi 展示我的所有书签；`~/.local/bin/bookmarks.py` 用于在命令行中管理书签。其实也可以在同一个文件中实现这两个功能，但是我有点懒了，写好就不想动它们了。

在 `~/.config/hypr/scripts/bookmark.py` 中，定义了数据库的位置为 `~/.config/hypr/scripts/bookmarks.db`，从我的环境变量中查找到 `BOOKMARKS_KEY` 的内容之后，作为密码揭秘数据库，将数据库的内容转换为"网站备注 | URL"的形式传给 wofi ，在 wofi 中选自之后使用 `xdg-open` 打开网址。

在 `~/.local/bin/bookmarks.py` 中，请求用户输入网址和网址备注，可以添加一个网址；输入 '-l' 参数可以列出所有书签；输入 '-d' 参数和网址备注可以删除一个书签。

最后使用 `chomd` 添加可执行权限，在开头添加 python 路径，即可直接运行。

# 实现

`~/.config/hypr/scripts/bookmark.py`: [pinme 链接](https://tvaby6xy.pinit.eth.limo)

`~/.local/bin/bookmarks.py`: [pinme 链接](https://puac7jwi.pinit.eth.limo)
