# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

`alear030.com` 的源码：一个纯静态的个人站点，目前是占位页，正式版还没做。站名取自作者的开源项目 [Alear030](https://github.com/Alear030/Alear030)（从零自研的 Python agent harness），视觉素材也来自那个仓库的 `assets/`。两个仓库各自独立：这里的提交不影响 harness，反过来也一样。

仓库是公开的。`public/` 是唯一的发布根，Cloudflare Pages 只发布这个目录；仓库根目录放的是说明类文件，不会出现在网站上。新增给访客看的文件放进 `public/`，其余都留在根目录。这样分是因为 Pages 会原样发布输出目录里的所有内容，放错了地方的文件，比如草稿或带密钥的配置，几分钟内就会被全网的扫描器读到。

没有构建步骤，没有依赖，HTML 是手写的。以后要正式写文章时，可以换成 Astro、Hugo 这类静态生成器，Pages 支持它们的自动构建。

## 上线链路

push 到 `main`，Cloudflare Pages 自动部署（项目名 `alear030`，没有构建命令，输出目录 `public`），几十秒后生效，测试地址是 `alear030.pages.dev`。

域名在 Dynadot 注册，DNS 托管在 Cloudflare，其余服务都挂在 Cloudflare 上。下面写的是每一项的用途，不是记录清单；线上实际是什么状态，以实时查询为准：

- 根域名和 `www` 都是 Pages 绑定自定义域名时自动建的、走代理的 CNAME，HTTPS 证书也是 Pages 自动签发的。`www` 由一条 Redirect Rule 301 跳转到根域名，路径和查询串都保留。添加这条规则时，Cloudflare 提示过「www 可能没开代理」，那是误报
- 邮件用的是 Email Routing：`hello@alear030.com` 转发到作者的个人邮箱，只收不发。catch-all 是关着的，发给不存在的地址会退信，不会被静默丢弃，这样写错地址的人能知道自己写错了
- MX、SPF 和 DKIM（`cf2024-1._domainkey`）都是 Email Routing 生成的，在 DNS 页面上是锁定状态。DMARC 设为 `p=reject`，因为这个域名目前不以自己的名义发信，从严不会误伤。以后要用 `@alear030.com` 发信，得先给发信服务配好 SPF 和 DKIM，再考虑放宽 DMARC
- 根域名上的 `google-site-verification` TXT 是 Search Console 的所有权凭证。删掉它，验证就会失效

## 设计语言

沿用 Alear030 项目的视觉：米色纸底 `#fef5e9`、深蓝灰 `#2b4a63`，背景铺一层很淡的六边形网格（对应项目吉祥物身上的六边形元素），整体留白多、很克制。站点上的名字写成全大写 `ALEAR030`，项目素材里则是 `Alear030`，两种写法都是有意的。

字体用的是系统字体栈，没有打包字体文件，所以不同设备上的字形和字距会有差异。

`public/og.jpg` 是分享卡片，从项目的 `assets/social-card.png` 压缩而来。卡片上的副标题介绍的是 harness 项目，不是个人站本身。做正式版时应该配一张专属卡片。

`404.html` 不能删。Cloudflare Pages 在发布目录里找不到 `404.html` 时，会对任何不存在的路径都返回首页加 200。这样扫描器去探测 `/wp-login.php` 这类路径，会误以为文件存在，然后反复来试探。

## 验证改动

本地没有服务器可起。看渲染效果，可以用无头浏览器截图，比如 Edge / Chrome 的 `--headless=new --screenshot`。要截完的是动画播放完毕的状态，就加上 `--virtual-time-budget`，因为首页的标题有淡入动画。注意无头 Chromium 的窗口最窄只能开到 500px 左右，`--window-size=390,...` 截出来的其实是 500px 宽的布局被裁了一截。要验证手机宽度，把页面套进指定宽度的 `<iframe>` 再截图。

上线后从外部验证：

- 页面状态：`curl -I https://alear030.com/`（应为 200）、`https://www.alear030.com/x?y=1`（应 301 跳到根域名，并保留路径和查询串）、任意不存在的路径（应为 404）
- DNS 记录：优先用 DNS over HTTPS 查，比如 `https://cloudflare-dns.com/dns-query?name=...&type=TXT`（请求头带 `accept: application/dns-json`）或 `https://dns.google/resolve?name=...&type=...`。明文 DNS 查询可能在网络中途被代理或运营商截下来代答，而且还会被缓存，返回的不一定是真实答案
- 邮件：测试信用另一个邮箱发，别用转发目标那个邮箱自己发给自己，Gmail 会把绕了一圈回来的信当成重复邮件。测试信写得太像垃圾邮件，比如全大写加一串感叹号，进垃圾箱是内容过滤的问题，跟配置无关。要看 SPF、DKIM、DMARC 的结果，打开那封信的原始邮件头
