---
title: "一条命令，把互联网上的任何内容变成 Markdown"
date: 2026-04-21T10:00:00+08:00
draft: false
tags: ["AI", "Agent", "Markdown", "Whisper", "工具"]
description: "微信公众号、YouTube、B站、抖音、小红书、网页、PDF——全部一条命令转成带图片的 Markdown。从被各平台封锁搞得想砸键盘，到彻底解决的全过程。"
---

## 一、我想要的很简单

我有一个习惯：看到好的内容就想存下来。不是收藏夹里吃灰的那种，是真的存成 Markdown，放进 Obsidian，能搜索、能引用、能和其他笔记串联。

这个习惯的好处不用多说——任何一个用 Obsidian 或者 Logseq 做知识管理的人都懂。你在 A 笔记里看到一个概念，双链到 B 笔记里三个月前记下的一段话，两个本来不相关的想法突然接通了。这种连接的价值，远大于把内容收藏在某个 APP 的文件夹里。

但现实是，**每个平台都在建围墙**。

微信公众号文章——图片防盗链，复制出来全是裂图。你精心存下来的一篇长文，三天后再看，满屏的灰色方块。文字是齐的，图片全没了。

B站视频——内容在视频里，文字版在哪？UP 主讲了四十分钟的技术方案分析，你想存下来当参考资料，对不起，你只能存一个链接。链接指向的是一个需要打开 APP 才能看的视频，你搜索不了里面的内容，引用不了里面的观点。

抖音、小红书——分享出来一串乱七八糟的文字加短链接。"7.92 复制打开抖音，看看【KNOCKZ街访的作品】财富再大，大不过健康..." 这串东西你存进 Obsidian 有什么用？

YouTube——先过一道 bot 检测再说。Google 对爬虫的防御一年比一年狠，2025 年底更新了一轮 JavaScript 挑战机制，大量工具直接失效。

我试过各种工具。浏览器插件 MarkDownload、SimpleRead、在线转换网站、Jina Reader、MarkItDown。每个都能解决一部分问题，但没有一个能全覆盖。尤其是视频内容，基本没人管——你把一个 B 站链接扔给任何一个"网页转 Markdown"工具，它最多给你抓到标题和简介，视频里讲了什么？不知道。

浏览器插件的另一个问题是：你得在浏览器里打开那个页面，手动点一下，等它跑完，再保存。这不叫自动化，这叫多了一步的手动操作。

我想要的其实很简单：**不管什么内容，给我一个链接，还我一份干净的 Markdown。**

带图片。带元数据。视频要有逐字稿。文章图片不能裂。一条命令，不需要打开浏览器，不需要手动操作，不需要登录任何平台。

既然没有现成的，就自己做一个。

## 二、先看结果

在讲技术之前，先看看最终做出来的东西长什么样。

### Web UI

`tomd --serve` 一条命令启动 Web 服务，浏览器自动打开。紫色渐变加玻璃态设计，简单到不需要说明书。

![Web UI 首页：粘贴链接、选类型、点转换](首页.png)

上面是文件拖拽区和 URL 输入框，下面一排平台类型选择——自动检测、网页、微信、YouTube、B站、抖音、小红书。大部分时候你不需要手动选类型，粘贴链接后直接点"开始转换"就行。

### 公众号文章转换

随便找一篇微信公众号文章，粘贴链接，点转换。几秒钟后，完整的 Markdown 就出来了——标题、作者、正文，所有图片都完好无损。

![微信公众号文章转换效果：图片完整保留，格式干净](公众号文章预览.png)

注意看图片：没有裂图。微信的图片防盗链被彻底绕过了——所有图片在转换时就被下载并转成了 base64 直接嵌进 Markdown 文件。这个 .md 文件是完全自包含的，你把它拖到任何地方打开，图片都在。

### 转换日志

切到"日志"标签，可以看到转换过程的每一步：类型检测、适配器选择、内容提取、完成。

![转换日志：清晰显示每一步的处理过程](日志.png)

这个日志在调试问题的时候特别有用。如果转换失败了，看一眼日志就知道卡在哪一步。

### 视频逐字稿

视频类内容是这个工具最与众不同的地方。粘贴一个 B 站视频链接，它会自动下载音频，用 Whisper ASR 转写成带时间戳的逐字稿。

![B站视频转换效果：自动生成带时间戳的逐字稿](视频逐字稿.png)

每一句话都标注了时间戳——`[00:00]`、`[00:01]`、`[00:04]`。你可以全文搜索视频里说了什么，可以引用某一句话并标注时间点。**视频里的信息终于可以被检索了。**

### Claude Code 集成

如果你是 Claude Code 用户，还有更方便的用法。这个工具已经发布为 ClawHub Skill，安装后可以直接在对话里用。

![ClawHub Skill 使用效果：在 Claude Code 对话中直接转换抖音视频](skill使用效果.png)

看这个截图：我直接把抖音的分享文本粘贴给 Claude——那一大串"7.92 复制打开抖音..."的文字，Claude 自动提取出 URL，调用 `tomd` 转换，几秒钟后返回完整的视频信息和 ASR 转写的对话内容。

这就是我想要的工作流：**看到什么内容，扔给工具，拿回 Markdown。** 不管内容在哪个平台，不管是文字还是视频，不管平台怎么防盗链、怎么反爬。

好了，结果看完了。下面讲讲这些东西是怎么做出来的。

## 三、技术选型：每个平台都是一场战争

做之前我以为最难的是格式转换。做完才知道，最难的是**从平台手里把内容拿出来**。

每个平台都有自己的围墙花园。每堵墙的砖都不一样——有的用防盗链，有的用 JavaScript 渲染，有的用 bot 检测，有的把数据藏在奇怪的地方。你以为找到了通用方案，下一个平台就给你当头一棒。

### 通用网页抓取：两层防线

通用网页用 [Trafilatura](https://github.com/adbar/trafilatura) 作为第一层。这是一个专门做网页正文提取的 Python 库，F1 分数 0.958，在同类工具里算最好的。纯 HTTP 请求，不需要浏览器，速度快。

但有些页面是 JavaScript 渲染的——React、Vue、Next.js 写的单页应用，Trafilatura 拿到的是一个空壳 HTML，里面啥内容都没有。

加了一层 [Crawl4AI](https://github.com/unclecode/crawl4ai) 兜底。Crawl4AI 是 Playwright 驱动的浏览器抓取方案，它会启动一个真正的 Chromium 浏览器去加载页面，等 JavaScript 跑完再提取内容。慢一些，但能覆盖 Trafilatura 搞不定的页面。

两层组合：先快后稳，覆盖了 95% 以上的网页。

**中间踩了个坑**，折腾了好几轮才解决。Trafilatura 提取出来的文本里，图片变成了 `Image: description` 这种纯文本占位符。位置是对的——图片确实应该出现在这个位置——但 URL 丢了。

一开始试过从 Trafilatura 的 XML 输出里找图片信息，没找到。又试过让 Trafilatura 输出 HTML 格式再解析，图片 URL 还是没有。

最后的方案有点"笨"但有效：从原始 HTML 的 `<article>` 区域（或者 `<main>`、`<div class="content">` 等常见正文容器）提取所有 `<img>` 标签，拿到 URL 列表。然后在 Trafilatura 的输出里找到所有 `Image:` 占位符，按顺序一一对应替换成 Markdown 图片语法 `![description](url)`。

这个方案假设图片在正文容器里的出现顺序和 Trafilatura 占位符的顺序一致——实测下来确实如此。不优雅，但管用。

### 微信公众号：和防盗链斗智斗勇

公众号文章本身不难抓，`mp.weixin.qq.com` 的页面结构很规整，HTML 直接请求就能拿到。难的是图片。

微信给所有图片加了 Referer 防盗链。当你在微信客户端或者微信内置浏览器里看文章时，请求头里有正确的 Referer，图片正常加载。但你把图片 URL 拷出来，在其他地方打开，返回的是一张 "此图片来自微信公众号，未经允许不可引用" 的占位图。

这意味着什么？你把公众号文章存成 Markdown，里面的图片链接全是微信的 CDN 地址。当天看没问题（浏览器可能有缓存），过两天再看——满屏裂图。

我评估了几个方案：

**方案一：加 Referer 头请求。** 理论上给 HTTP 请求加上 `Referer: https://mp.weixin.qq.com/` 就能绕过。但微信还会校验其他参数，比如 cookie 和时效性签名。不稳定。

**方案二：用微信公众号镜像站。** 有些第三方服务会缓存公众号文章和图片。作为备用方案可以，但不能作为主方案——镜像站随时可能挂。

**方案三：下载图片，转 base64，直接嵌进 Markdown。** 在抓取时就把图片下载到本地，转成 base64 编码的 Data URI，直接写进 Markdown 文件里。

```markdown
![图片描述](data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...)
```

最终选了方案三。生成的 .md 文件会大一些（一张图几十 KB 到几百 KB，base64 编码后还要再大 33%），但**永远不会裂图**。因为图片数据就在文件里面，不依赖任何外部链接。你把这个文件拷到 U 盘里，十年后打开，图片还在。

这才是"存下来"应该有的样子。

### 小红书：从一堆垃圾文本里抠出 URL

小红书的技术方案本身不复杂——页面的所有数据都藏在 `window.__INITIAL_STATE__` 这个 JavaScript 变量里。纯 HTTP 请求拿到 HTML，正则提取 JSON，解析出标题、正文、图片 URL。零依赖，不需要 cookie，不需要浏览器。

但小红书有个**非技术问题**让我很头疼：用户分享出来的不是干净的 URL。

当你在小红书 APP 里点"分享"再点"复制链接"，剪贴板里拿到的是这么一段东西：

```
照着做！秦岭AI白客松... 🔗 财富再大，大不过健康；有健康，才有一切！ 复制此链接打开【小红书】，直接查看内容！ https://xhslink.com/a/xxxxx
```

一大段推广文案，真正有用的 URL 藏在最后面。如果不做处理，用户粘贴进来的是一整段文字，工具根本不知道你要转换什么。

最后写了一个 URL 提取函数——用正则从分享文本里把 `https://` 开头的链接抠出来。看起来毫无技术含量，但不加这个函数，工具对 80% 的小红书用户来说就没法用。因为没有人会手动去提取那段链接。

这种"脏活"在整个项目里到处都是。技术上没难度，但不做就是不好用。

### 抖音：`None` vs `{}` 的经典教训

抖音的技术路线和小红书类似——数据藏在 `window._ROUTER_DATA` 这个 JavaScript 变量里。纯 HTTP 请求，正则提取 JSON，解析出标题、描述、视频流地址。

但调试的时候踩了一个 Python 的经典坑，排查了半小时。

代码里有一段链式字典访问：

```python
data = json_data.get("loaderData", {}).get("video_(id)/page", {}).get("videoInfoRes", {})
```

看起来很安全对吧？每一层都有默认值 `{}`，理论上不管哪个 key 不存在都不会报错。

但问题出在：**当 JSON 里某个 key 的值是 `None` 时**。

```python
>>> {"key": None}.get("key", {})
None
```

`dict.get("key", {})` 的意思是"如果 key 不存在，返回 `{}`"。但 `None` 不是"不存在"——key 存在，值就是 `None`。所以 `.get()` 返回的是 `None` 本身，而不是默认值 `{}`。

下一个 `.get()` 就炸了：`None.get("sub_key")` → `AttributeError`。

解决方案很简单，写了一个 `_safe()` 函数：

```python
def _safe(val, default=None):
    return val if val is not None else (default if default is not None else {})
```

每次取值都过一遍 `_safe()`。丑但管用。

这种 bug 在 JSON 解析的场景里特别常见——你不知道上游 API 返回的某个字段会不会是 `null`（对应 Python 的 `None`），防御性编程必须做到位。

## 四、YouTube：最折腾的一个平台

YouTube 值得单独开一章来讲，因为它贡献了这个项目里 60% 的调试时间。

### 第一道关：bot 检测和 Deno

从 2025 年 11 月开始，YouTube 更新了反爬机制，加入了新的 JavaScript 挑战。[yt-dlp](https://github.com/yt-dlp/yt-dlp) 为了应对这个变化，引入了一个新的依赖——[Deno](https://deno.com/)。yt-dlp 会调用 Deno 来执行 YouTube 的 JavaScript 挑战代码，解出 bot 检测的验证令牌。

安装 Deno 之后，在终端里直接跑 yt-dlp，一切正常。

但是，在 Python 的 `subprocess` 里调用 yt-dlp，死活不行。

报错信息是 `"Only images are available for download"`——这是 YouTube 没有认可你的请求时的典型回应。换一种说法就是：YouTube 觉得你是 bot，拒绝给你视频和音频。

**这个 bug 调了很久。**

同一条 yt-dlp 命令，在终端里跑正常，在 `subprocess.run()` 里跑就失败。环境变量检查了，PATH 检查了，工作目录检查了，都一样。

最后是对比两个环境的 stderr 输出才发现差异。终端里跑的时候，stderr 有一行 `[youtube] Extracting with PhantomJS player` 之类的日志，说明 Deno JS 求解器生效了。而 subprocess 里没有这行。

原因是：yt-dlp 在终端环境里会缓存 Deno 下载的 JavaScript 求解器脚本。第一次运行后，后续运行直接用缓存。但 subprocess 创建的进程环境里找不到这个缓存路径。

**解决方案：** 给 yt-dlp 加上 `--remote-components ejs:github` 参数，强制从 GitHub 下载 JavaScript 求解器组件，不依赖本地缓存。

```python
def _ytdlp_cmd(with_cookies: bool = False) -> list[str]:
    base = ["yt-dlp", "--remote-components", "ejs:github"]
    if with_cookies:
        import shutil
        for browser in ["chrome", "safari", "firefox"]:
            if shutil.which(browser) or browser == "chrome":
                return base + ["--cookies-from-browser", browser]
    return base
```

这一个参数，卡了我整整一个下午。

### 第二道关：字幕获取的三层策略

YouTube 视频的文字内容获取，我设计了三层策略：

**第一层：yt-dlp 字幕提取。** 很多 YouTube 视频有上传者提供的字幕（手动或自动生成的）。yt-dlp 的 `--write-subs` 和 `--write-auto-subs` 可以下载字幕文件。字幕格式五花八门——json3、vtt、srt、srv3——每种格式写了一个解析器。

json3 格式最好处理，是标准 JSON，带时间戳和文本。vtt 和 srt 需要用正则匹配时间戳行和文本行，还要注意去重（自动字幕经常有重复行）。srv3 是 XML 格式，用 ElementTree 解析。

**第二层：下载音频 + ASR 转写。** 如果没有字幕，就下载视频的音频流，用 Whisper 做语音转文字。yt-dlp 的 `-f bestaudio` 参数可以只下载音频，不下载视频，节省时间和带宽。

**这两层之间的切换是自动的。** 先尝试拿字幕，拿不到就走 ASR。用户不需要关心视频有没有字幕——工具自己处理。

### 第三道关：cookie 的优雅降级

部分 YouTube 视频需要登录才能访问——年龄限制内容、地区限制内容、或者 YouTube 判定你需要登录才给访问的内容。

最初的实现是把 cookie 作为必选依赖——你必须先登录 Chrome 的 Google 账号，工具才能工作。但实测下来，**大部分视频不需要登录**。把 cookie 作为必选项会吓跑一大批用户。

最终的策略是**优雅降级**：

```python
def _run_ytdlp(args, verbose=False, timeout=60):
    # 先不带 cookie 试
    cmd = _ytdlp_cmd(with_cookies=False) + args
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)

    if result.returncode == 0:
        return result

    # 被 bot 拦截了？自动加 cookie 重试
    if "Sign in to confirm" in result.stderr or "bot" in result.stderr.lower():
        if verbose:
            print("[youtube] Bot detected, retrying with browser cookies...")
        cmd = _ytdlp_cmd(with_cookies=True) + args
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=timeout)

    return result
```

先不带 cookie 试。如果成功了，皆大欢喜。如果被拦了（stderr 里出现 "Sign in to confirm" 或 "bot"），自动从 Chrome 浏览器读取 cookie 重试。

这个设计的好处：用户第一次使用不需要任何额外配置。大部分视频直接就能转换。只有少数需要登录的视频，工具会自动尝试读取浏览器 cookie——前提是你在 Chrome 里登录过 Google。如果连 cookie 都不行，才会返回错误。

**最大限度降低用户的使用门槛。**

### B 站：相对省心

B站是这批平台里最省心的一个。它的 Web API 不需要登录就能拿到视频元数据——标题、UP 主、描述、时长、播放量。

音频流用 DASH 格式提供，从 API 拿到音频的 CDN 地址后直接下载。下载下来是 m4s 格式，用 ffmpeg 转成标准音频格式，再喂给 Whisper 转写。

有一个小细节：B 站的视频 URL 经常带一堆查询参数，比如 `?spm_id_from=333.1007.tianma.11-1-39.click`。这些参数对访问内容没用，但如果不清理掉，有时候会影响 API 调用。加了一个 URL 清洗步骤，只保留 BV 号。

## 五、ASR：让视频开口说话

视频平台的内容大部分没有文字版。YouTube 有些有自动字幕，但质量参差不齐；B站没有公开的字幕接口；抖音更不用说。

要把视频变成 Markdown，核心能力是 **ASR（Automatic Speech Recognition，自动语音识别）**。

用的是 OpenAI 的 [Whisper](https://github.com/openai/whisper) turbo 模型。为什么选 Whisper？几个原因：

1. **开源免费**，本地运行，不需要 API key，不依赖网络
2. **多语言支持**，中文、英文、日文都能转，不用操心语言检测
3. **带时间戳输出**，每句话标注开始和结束时间
4. **效果够用**，虽然不是最顶尖的，但日常内容的转写准确率完全够

通过 `brew install openai-whisper` 安装，CPU 模式运行。在我的 Mac 上，一段 4 分钟的视频大约 1 分钟完成转写。不算快，但可以接受。

转写流程：

```
视频/音频 URL
  ↓ yt-dlp 下载音频流
  ↓ ffmpeg 提取/转换音频格式
  ↓ Whisper turbo 模型转写
  ↓ 解析时间戳 + 文本
  ↓ 格式化为 Markdown 逐字稿
```

输出的逐字稿长这样：

```markdown
## Transcript

Duration: 01:57

**[00:00]** 有一个人前来买瓜
**[00:01]** 今天肯定生意行
**[00:04]** 我有感觉
**[00:05]** 哥们
**[00:06]** 你这瓜多少钱一斤呢
```

每行一个时间戳，一句话。你可以在 Obsidian 里全文搜索视频内容，也可以引用某句话并标注它出现在视频的第几秒。

后续可以升级到 [whisper.cpp](https://github.com/ggerganov/whisper.cpp)——纯 C++ 实现，支持 Apple Silicon 的 Metal 加速，在 M 系列芯片上比 Python 版快 3-5 倍。但目前 CPU 模式够用，先不折腾。

## 六、图片处理：base64 内嵌方案

图文类内容的另一个核心问题是图片。

传统的 Markdown 图片语法是引用外部 URL：

```markdown
![图片描述](https://cdn.example.com/img/xxx.png)
```

这在写文档时没问题。但用来**存档**就有风险——URL 可能失效、CDN 可能下线、防盗链可能拦截。尤其是微信公众号的图片，你存下来的 URL 过两天就打不开了。

我的方案是**全部转 base64 内嵌**：

1. 抓取文章时，解析出所有图片 URL
2. 下载图片到内存
3. 转成 base64 编码
4. 用 Data URI 替换原始 URL

```markdown
![图片描述](data:image/jpeg;base64,/9j/4AAQSkZJRg...)
```

这样生成的 Markdown 文件是**完全自包含的**——一个 .md 文件包含了所有内容，文字和图片都在里面。你可以：

- 把文件拷贝到任何设备，图片都完好
- 拖到 Obsidian 里直接渲染，不需要额外操作
- 十年后打开，图片还在
- 离线环境下查看，不需要网络

代价是文件体积增大。一篇有 10 张图的文章，.md 文件可能有几 MB。但存储空间不值钱，图片裂掉的心理成本比磁盘空间贵多了。

这个决策是从微信公众号的防盗链被逼出来的，但最终我把它推广到了所有图文类内容。**不给任何平台裂图的机会。**

## 七、Web UI：不是所有人都爱命令行

CLI 做完了，自己用没问题。`tomd <url>` 一敲回车，几秒钟后 .md 文件出现在当前目录。

但要给别人用——或者自己在某些场景下快速预览效果——还是需要一个界面。

用 FastAPI 写了一个 Web 服务。后端三个 API：

- `POST /api/convert`：接收 URL 和类型参数，启动异步转换任务
- `POST /api/upload`：接收上传文件，启动转换
- `GET /api/status/{id}`：轮询任务状态，获取日志和结果

前端是一个单文件 HTML——所有 CSS 和 JavaScript 内嵌在 `index.html` 里，零构建步骤，零外部依赖。不需要 npm、不需要 webpack、不需要 React。一个 HTML 文件解决所有前端需求。

设计风格用了紫色渐变背景加玻璃态（glassmorphism）卡片。看起来不像一个开发者工具，倒像一个产品。但这不重要——重要的是好用。

功能：

- **粘贴 URL 或分享文本**：自动识别链接，不需要手动提取
- **拖拽上传本地文件**：PDF、DOCX、PPTX 等，拖进来就开始转换
- **实时日志**：转换过程中可以看到每一步的进度
- **三种查看模式**：预览（渲染后的 Markdown）、源码（原始 Markdown 文本）、日志
- **一键复制 / 下载**：复制 Markdown 到剪贴板，或下载 .md 文件
- **会话历史**：当前会话的所有转换记录，随时回看

启动方式：

```bash
tomd --serve              # 默认 8765 端口
tomd --serve --port 9000  # 自定义端口
```

服务启动后浏览器自动打开。默认只监听 `127.0.0.1`，不会暴露到网络上。

## 八、整体架构

最终的系统架构很清晰：

```
输入 (URL / 文件 / 分享文本)
  ↓ 自动识别类型（detection.py）
  ↓ 路由到对应平台适配器（adapters/*.py）
  ↓ 视频类：字幕优先，无字幕则下载音频 → Whisper ASR 转写
  ↓ 图文类：提取正文 + 下载图片内嵌 base64
  ↓ 组装 Markdown：YAML frontmatter + 正文（formatter.py）
  ↓
输出 .md 文件
```

每个平台一个适配器，各管各的：

```
src/anything_to_md/
├── cli.py              # CLI 入口（tomd 命令）
├── web.py              # FastAPI Web UI 服务
├── detection.py        # 输入类型自动识别
├── pipeline.py         # 主编排流程
├── models.py           # 数据模型
├── formatter.py        # YAML frontmatter + MD 组装
├── asr.py              # ASR 引擎封装（Whisper）
└── adapters/
    ├── webpage.py       # 通用网页（Trafilatura → Crawl4AI）
    ├── wechat.py        # 微信公众号
    ├── youtube.py       # YouTube（yt-dlp + Deno + ASR）
    ├── bilibili.py      # B站
    ├── douyin.py        # 抖音（iesdouyin）
    ├── xiaohongshu.py   # 小红书
    └── file.py          # 本地文件（MarkItDown）
```

`detection.py` 负责从输入（URL、文件路径、分享文本）自动判断类型。判断逻辑很简单——主要是域名匹配：

- `mp.weixin.qq.com` → 微信
- `youtube.com` / `youtu.be` → YouTube
- `bilibili.com` → B站
- `douyin.com` / `v.douyin.com` → 抖音
- `xiaohongshu.com` / `xhslink.com` → 小红书
- 文件路径 → 本地文件
- 其他 URL → 通用网页

`pipeline.py` 是编排器，根据检测结果选择适配器，调用适配器的 `extract()` 方法，拿到结构化数据，再调用 `formatter.py` 组装成最终的 Markdown。

输出的 Markdown 格式统一：

```markdown
---
title: 文章标题
author: 作者
source_url: https://原始链接
source_type: webpage
date: 2026-04-22
---

正文内容...
```

YAML frontmatter 包含元数据——标题、作者、来源 URL、类型、日期。正文是干净的 Markdown。视频类内容会在正文后面追加 `## Transcript` 部分。

支持的平台汇总：

| 平台 | 方案 | Cookie |
|------|------|--------|
| 网页 | Trafilatura + Crawl4AI | 不需要 |
| 微信公众号 | 直接抓取 + 镜像站回退 | 不需要 |
| YouTube | yt-dlp + Deno + Whisper ASR | 自动读取 |
| B站 | Web API + DASH 音频 + Whisper ASR | 不需要 |
| 抖音 | iesdouyin 零依赖方案 + Whisper ASR | 不需要 |
| 小红书 | \_\_INITIAL\_STATE\_\_ 解析 | 不需要 |
| PDF/Office | MarkItDown | - |

## 九、安装和使用

### 安装

```bash
# 基础安装（网页、微信、本地文件）
uv pip install -e "."

# 含视频支持（YouTube、B站、抖音）
uv pip install -e ".[video]"

# 全部安装（含 Web UI）
uv pip install -e ".[all]"
```

系统依赖：

**macOS (Homebrew)**

```bash
brew install yt-dlp deno ffmpeg openai-whisper
```

**Linux (Ubuntu/Debian)**

```bash
pip install yt-dlp
curl -fsSL https://deno.land/install.sh | sh
sudo apt install ffmpeg
pip install openai-whisper
```

**Windows (Scoop)**

```powershell
scoop install yt-dlp deno ffmpeg
pip install openai-whisper
```

### 命令行使用

```bash
# 网页
tomd https://example.com/article

# 微信公众号
tomd "https://mp.weixin.qq.com/s/abc123"

# YouTube
tomd "https://www.youtube.com/watch?v=xxxxx"

# B站
tomd "https://www.bilibili.com/video/BVxxx"

# 抖音（支持直接粘贴分享文本）
tomd "https://v.douyin.com/xxx"

# 小红书
tomd "https://www.xiaohongshu.com/explore/xxx"

# 本地 PDF
tomd ~/Documents/paper.pdf

# Web UI
tomd --serve
```

### ClawHub Skill

Claude Code 用户可以直接安装 Skill 使用：

```bash
clawhub install anything-to-md
```

安装后在对话里说"帮我把这个链接转成 Markdown"，Agent 会自动调用 `tomd` 完成转换。

项目开源在 GitHub：[haiwenai/anything-to-md](https://github.com/haiwenai/anything-to-md)

## 十、调试日记：那些让我想砸键盘的时刻

这个项目从第一行代码到全平台跑通，断断续续折腾了一周。大部分时间不是在写代码，是在和各个平台的反爬机制斗智斗勇。

记几个印象深刻的坑：

### YouTube 的 subprocess 之谜

终端里跑 yt-dlp 一切正常。放到 Python `subprocess.run()` 里就报 "Only images are available for download"。

同一条命令，同一台机器，同一个用户。唯一的区别是运行环境。

我花了一个下午逐项对比两个环境的差异。PATH 一样。环境变量一样。工作目录一样。权限一样。

最后是逐行对比两份 stderr 输出才发现——终端环境里 yt-dlp 使用了缓存的 Deno JS 求解器脚本，subprocess 环境里没有这个缓存。加上 `--remote-components ejs:github` 参数后一切正常。

**教训：** 终端环境和 subprocess 环境并不完全相同，即使你觉得"都一样"。当遇到"终端里正常但代码里不行"的 bug，第一件事应该是对比两个环境的完整输出，而不是反复调试代码逻辑。

### 抖音的 `None` 陷阱

```python
{"key": None}.get("key", {})  # 返回 None，不是 {}
```

Python 社区争论过这个设计是否合理。不管合不合理，你得防着它。调试这个 bug 花了半小时——因为报错信息是 `AttributeError: 'NoneType' object has no attribute 'get'`，指向的是链式调用的第二个 `.get()`，你以为是第二个 key 不存在，其实是第一个 key 的值是 `None`。

### 微信图片的持久战

一开始用的是 Referer 伪造方案——设置请求头 `Referer: https://mp.weixin.qq.com/`。能用，但不稳定。有时候能拿到图片，有时候返回 403。

然后试了微信公众号镜像站——确实能绕过防盗链，但镜像站本身不够稳定，而且有些文章镜像站没有收录。

最后改成了 base64 内嵌方案。一劳永逸。代价是文件大一些，但我可以接受"一个 5MB 的 .md 文件"，不能接受"一个图片全裂的 .md 文件"。

### B 站 URL 清洗

用户给一个 B 站链接：`https://www.bilibili.com/video/BV1TEQbBsEPX?spm_id_from=333.1007.tianma.11-1-39.click`

带着一大堆查询参数。直接拿这个 URL 去调 API，有时候正常，有时候报错。问题在于 `spm_id_from` 之类的追踪参数干扰了 API 的解析。

解决方案：URL 预处理，只保留路径部分的 BV 号，扔掉所有查询参数。简单但必要。

### 小红书分享文本的"人性化"处理

技术上用正则提取 URL 很简单。但要考虑到各种边界情况：

- 有的分享文本 URL 前面有中文逗号，没有空格
- 有的 URL 用的是短链（`xhslink.com`），需要跟随重定向拿到真实 URL
- 有的分享文本里有多个 URL（正文一个，评论一个），需要取第一个

每个边界情况都不难处理，但你得真的遇到了才知道需要处理。这就是为什么这种工具必须自己用、反复用、用真实数据测，才能做到好用。

## 十一、回头看

项目做完了。所有平台测试通过。CLI 和 Web UI 都能用。ClawHub Skill 也发布了。

但最让我满意的不是技术实现——而是一个工作流的改变。

以前看到一篇好的公众号文章，我的流程是：复制全文 → 粘贴到 Obsidian → 手动修格式 → 发现图片裂了 → 手动下载图片 → 手动插入图片 → 改了十分钟 → 算了不存了。

以前看到一个有料的 B 站视频，我的流程是：收藏 → 扔进收藏夹吃灰 → 半年后想起来 → 视频被删了。

现在的流程是：复制链接 → `tomd <链接>` → 完事。一条命令，十秒钟。Markdown 文件自动保存，图片完好，视频有逐字稿，元数据齐全。拖进 Obsidian，和其他笔记互相链接。

**信息的保存效率提升了不止十倍，但最大的变化是心理门槛消失了。** 以前因为"存下来太麻烦"而放弃保存的内容，现在随手一存。积累多了，笔记库里的连接就越来越多，新想法的冒出就越来越快。

每个平台都有自己的围墙，每堵墙的砖都不一样。

但翻过去之后，所有内容都变成了同一种格式：**干净的 Markdown**。存进 Obsidian，全文搜索，互相引用，永久保存。

这才是我想要的互联网的样子。

---

*项目开源在 GitHub：[haiwenai/anything-to-md](https://github.com/haiwenai/anything-to-md)。MIT 协议，自由使用。如果你也被围墙花园搞得想砸键盘，试试看。*
