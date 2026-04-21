---
title: "一条命令，把互联网上的任何内容变成 Markdown"
date: 2026-04-21T10:00:00+08:00
draft: false
tags: ["AI", "Agent", "Markdown", "Whisper", "工具"]
description: "微信公众号、YouTube、B站、抖音、小红书、网页、PDF——全部一条命令转成带图片的 Markdown。从被各平台封锁搞得想砸键盘，到彻底解决的全过程。"
---

## 一、我想要的很简单

我有一个习惯：看到好的内容就想存下来。不是收藏夹里吃灰的那种，是真的存成 Markdown，放进 Obsidian，能搜索、能引用、能和其他笔记串联。

但现实是，每个平台都在建围墙。

微信公众号文章——图片防盗链，复制出来全是裂图。B站视频——内容在视频里，文字版在哪？抖音、小红书——分享出来一串乱七八糟的文字加短链接。YouTube——先过一道 bot 检测再说。

我试过各种工具：浏览器插件、在线转换网站、Jina Reader、MarkDownload。每个都能解决一部分问题，但没有一个能全覆盖。尤其是视频内容，基本没人管。

我想要的其实很简单：**不管什么内容，给我一个链接，还我一份干净的 Markdown。**

带图片。带元数据。视频要有逐字稿。文章图片不能裂。

既然没有现成的，就自己做一个。

## 二、技术选型：每个平台都是一场战争

做之前我以为最难的是格式转换。做完才知道，最难的是**从平台手里把内容拿出来**。

### 网页抓取

通用网页用 [Trafilatura](https://github.com/adbar/trafilatura) 就够了，F1 0.958，纯 HTTP 请求，速度快。但有些页面是 JavaScript 渲染的，Trafilatura 拿到的是空壳。

加了一层 [Crawl4AI](https://github.com/unclecode/crawl4ai) 兜底——Playwright 驱动的浏览器抓取。两层组合覆盖了 95% 以上的网页。

中间踩了个坑：Trafilatura 提取出来的文本里图片变成了 `Image: description` 这种占位符，位置是对的但 URL 丢了。折腾了好几轮，最后的方案是从原始 HTML 的 `<article>` 区域提取 `<img>` 标签，按顺序一一对应替换占位符。

### 微信公众号

公众号文章本身不难抓，难的是图片。微信给所有图片加了 Referer 防盗链，你把 Markdown 存下来，图片全是裂的。

解决方案：下载图片，转 base64，直接嵌进 Markdown 文件。生成一个完全自包含的 .md 文件，不依赖任何外部链接。文件会大一些，但**永远不会裂图**。

### 小红书和抖音

这两个平台的数据藏在页面的 JavaScript 变量里。小红书是 `window.__INITIAL_STATE__`，抖音是 `window._ROUTER_DATA`。

纯 urllib 请求拿到 HTML，正则提取 JSON，解析出标题、正文、图片 URL、视频流地址。零依赖，不需要 cookie，不需要浏览器。

小红书有个细节：用户分享出来的不是干净的 URL，而是一大段文字——"照着做！秦岭AI白客松... 复制此链接打开【小红书】..."。得先用正则把 URL 从分享文本里抠出来。

抖音踩了一个 Python 的经典坑：`dict.get("key", {}).get("sub_key")`——当 JSON 里 `key` 的值是 `None` 时，`.get("key", {})` 返回的是 `None` 而不是 `{}`，因为 `None` 不是"没有这个 key"。写了一个 `_safe()` 函数兜底。

### YouTube

YouTube 是最折腾的一个。

**第一道关：bot 检测。** 从 2025 年 11 月开始，yt-dlp 必须配合 [Deno](https://deno.com/) 来解 YouTube 的 JavaScript 挑战。而且在 Python 的 `subprocess` 里运行时，还需要加 `--remote-components ejs:github` 参数——因为 subprocess 环境里找不到 JS 挑战求解器的缓存。

这个 bug 调了很久。直接在终端跑 yt-dlp 一切正常，放到代码里就报 "Only images are available for download"。最后对比两个环境的 stderr 才发现差异。

**第二道关：字幕获取。** 有字幕的视频好办，yt-dlp 的 `--write-subs` 和 `--write-auto-subs` 可以拿到。没字幕的视频，下载音频，跑 Whisper ASR 转文字。

**第三道关：cookie。** 部分视频不登录根本访问不了。最后的策略是先不带 cookie 试，被拦了自动从 Chrome 浏览器读取 cookie 重试。大部分视频不需要登录，需要的时候自动处理。

### B站

相对省心。B站的 Web API 不需要登录就能拿到元数据和 DASH 音频流。下载音频，Whisper 转写，搞定。

## 三、ASR：让视频开口说话

视频平台的内容大部分没有文字版。要把视频变成 Markdown，核心是 **ASR（语音转文字）**。

用的是 OpenAI 的 [Whisper](https://github.com/openai/whisper) turbo 模型，通过 brew 安装，CPU 模式运行。一段 4 分钟的视频大约 1 分钟完成转写，效果够用。

流程：下载视频/音频 → ffmpeg 提取音频 → Whisper 转写 → 带时间戳的逐字稿。

后续可以升级到 whisper.cpp（Metal 加速，Apple Silicon 上快 3 倍以上），但目前 CPU 模式也够用。

## 四、Web UI：不是所有人都爱命令行

CLI 做完了，自己用没问题。但要给别人用，或者自己快速预览效果，还是需要一个界面。

用 FastAPI 写了一个 Web 服务，单文件 HTML 前端，紫色渐变加玻璃态设计。`tomd --serve` 一条命令启动，浏览器自动打开。

粘贴 URL、拖拽文件、选类型、点转换。实时显示日志，转换完成后可以预览渲染效果、查看源码、一键复制或下载 .md 文件。

所有图片在服务端就转成了 base64 嵌入 Markdown，下载下来的 .md 文件是完全自包含的，拖到 Obsidian 里直接能看。

## 五、最终的架构

```
输入 (URL / 文件 / 分享文本)
  ↓ 自动识别类型
  ↓ 对应平台适配器提取内容
  ↓ 视频类：字幕优先，无字幕则下载音频 → Whisper ASR 转写
  ↓ 图文类：提取正文 + 下载图片内嵌 base64
  ↓ 组装 Markdown (YAML frontmatter + 正文)
  ↓
输出 .md 文件
```

支持的平台：

| 平台 | 方案 | Cookie |
|------|------|--------|
| 网页 | Trafilatura + Crawl4AI | 不需要 |
| 微信公众号 | 直接抓取 + 镜像站回退 | 不需要 |
| YouTube | yt-dlp + Deno + Whisper ASR | 自动读取 |
| B站 | Web API + DASH 音频 + Whisper ASR | 不需要 |
| 抖音 | iesdouyin 零依赖方案 + Whisper ASR | 不需要 |
| 小红书 | \_\_INITIAL\_STATE\_\_ 解析 | 不需要 |
| PDF/Office | MarkItDown | - |

## 六、用起来

```bash
# 安装
uv pip install -e ".[all]"
brew install yt-dlp deno ffmpeg openai-whisper

# 命令行
tomd https://example.com/article
tomd "https://www.youtube.com/watch?v=xxxxx"
tomd "https://mp.weixin.qq.com/s/abc123"
tomd ~/Documents/paper.pdf

# Web UI
tomd --serve
```

项目开源在 GitHub：[haiwenai/anything-to-md](https://github.com/haiwenai/anything-to-md)

ClawHub Skill 也已发布，Claude Code 用户可以直接安装：

```bash
clawhub install anything-to-md
```

## 七、回头看

这个项目从第一行代码到全平台跑通，断断续续折腾了一周。大部分时间不是在写代码，是在和各个平台的反爬机制斗智斗勇。

抖音的 `None` vs `{}` 让我排查了半小时。YouTube 的 subprocess 环境差异让我对着两份 stderr 逐行 diff。微信的防盗链让我从远程代理方案一路改到 base64 内嵌。小红书的分享文本让我加了一个看起来毫无技术含量但不加就没法用的 URL 提取函数。

每个平台都有自己的围墙，每堵墙的砖都不一样。

但翻过去之后，所有内容都变成了同一种格式：**干净的 Markdown**。存进 Obsidian，全文搜索，互相引用，永久保存。

这才是我想要的互联网的样子。
