---
layout: post
title: 一种基于 qutebrowser 的 1 秒跳过 youtube 广告的方法
date: 2026-01-01
author: SL
header-img: img/post-bg-debug.png
catalog: true
tags: -- qutebrowser
  -- youtube
  -- 油管
  -- 广告
---

# 背景

随着 YouTube 不断强化对广告拦截器的检测机制，用户被迫观看广告已成为常态。尽管社区提出了多种绕过广告的方案，但绝大多数方法已因平台的持续更新而失效。

此前使用 zen-browser 配合 vimium C 插件时，其点击功能无法有效识别并点击广告跳过按钮，导致每次都需要手动等待 5 秒后使用鼠标操作。

近期转向 qutebrowser 后，发现其内置的 hint 功能能够准确识别广告跳过按钮。基于这一特性，本文提出一种利用 qutebrowser hint 机制实现秒级跳过 YouTube 广告的自动化方法。

# 实现原理

qutebrowser 支持用户脚本功能，允许脚本与浏览器进行双向通信——既能从浏览器获取信息，也能向浏览器发送指令。通过 jseval 命令，可以在页面上下文中执行 JavaScript 代码，实现如加速广告播放等自定义逻辑。

利用 hint 命令可模拟鼠标点击操作，其 -f 参数能够自动点击页面上匹配的第一个元素。通过配置自定义的 hints.selector，可以精确限定 hint 功能仅识别和选择广告跳过按钮，从而实现自动化跳过。

# 实现过程

## 第一步：添加 hints.selector

在 config.py 中添加一行：

```python
c.hints.selectors["ytskip"] = [".ytp-skip-ad-button.ytp-ad-component--clickable"] # hint the youtube skip ad button
```

这一行的意思是：添加一个 hints.selector，内容为查找一个同时有 "ytp-skip-ad-button" 和 "ytp-ad-component--clickable" 两个组名的元素，满足条件的仅有 youtube 的跳过广告按钮。

## 第二步：添加速度控制功能

由于视频速度控制是一个比较常用的功能，所以我单独写了一个用户脚本，方便自己日常使用。

在 `.config/qutebrowser/userscripts/` 中添加名为 `qute-speed` 的文件，内容为：

```python
#!/usr/bin/env python3
import os, sys, textwrap

fifo = os.getenv('QUTE_FIFO')
if not fifo:
    sys.exit()

speed = float(sys.argv[1]) if len(sys.argv) > 1 else 2.0

js_multiline = textwrap.dedent(f'''\
    (function(speed){{
        const vids = document.querySelectorAll('video');
        let ok = 0;
        vids.forEach(v=>{{
            try{{ v.playbackRate = speed; ok++; }}
            catch(e){{}}
        }});
        if(ok) console.info('[speed-video] 已设置 '+ok+' 个 video → ×'+speed);
        else   console.warn('[speed-video] 未找到 <video> 元素');
        console.log("[speed-video] 速度设置为 "+speed);
    }})({speed});
''')

js_oneline = ' '.join(js_multiline.split()) # 由于 jseval 命令只能运行单行，所以这里压缩一下

with open(fifo, 'w') as f:
    f.write(f'jseval {js_oneline}\n')

with open(fifo, 'w') as f:
    f.write(f'message-info "视频已设为 ×{speed}"')
```

记得用 `chmod` 给脚本添加可执行权限。

现在就可以在 qutebrowser 中使用 `:spawn -u qute-speed` 调用这个脚本了。

## 第三步：用于跳过广告的用户脚本

在 `.config/qutebrowser/userscripts/` 中添加名为 `qute-ytad` 的文件，内容为：

```python
#!/bin/env python3

import os
import time

qute_fifo = os.environ["QUTE_FIFO"]
if not qute_fifo:
    exit(1)

def speed_control(speed: int):
    with open(qute_fifo, 'w') as f:
        f.write(f"spawn -u qute-speed {speed}\n")


speed_control(10)
time.sleep(1) # 等待 0.5 秒的话按钮不会立刻出现

speed_control(1)
with open(qute_fifo, 'w') as f:
    f.write("hint -f ytskip\n")
```

之后将脚本映射为快捷键即可
