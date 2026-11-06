# 访客统计（Cloudflare Web Analytics）

本站是 GitHub Pages 静态站点，自定义域名 `onlinechoice.online`。
访客统计用 **Cloudflare Web Analytics**，由 Cloudflare 在边缘自动注入 beacon。

**本仓库里没有任何统计代码，也不需要有。** 见下方「不要手动加 beacon」。

## 当前架构

```
访客 → Cloudflare 边缘（代理，注入 beacon）→ GitHub Pages（源站）
```

- 域名注册商：Crazy Domains（只负责注册，NS 已指向 Cloudflare）
- DNS 权威：Cloudflare（Free 方案）
- 源站：GitHub Pages，`main` 分支根目录
- SSL/TLS 模式：Full (strict)

## DNS 记录清单

全部在 Cloudflare 管理。Crazy Domains 那边的 DNS 已失效，不要在那里改任何东西。

| 类型 | 名称 | 内容 | 代理 |
|---|---|---|---|
| A ×4 | `onlinechoice.online` | `185.199.108.153` / `.109.153` / `.110.153` / `.111.153` | 橙云 |
| AAAA ×4 | `onlinechoice.online` | `2606:50c0:8000::153` / `8001::153` / `8002::153` / `8003::153` | 橙云 |
| CNAME | `www` | `jzhyzhy.github.io` | 橙云 |
| TXT | `_github-pages-challenge-jzhyzhy` | GitHub 域名归属验证串 | 不可代理 |

TXT 那条是 GitHub 验证域名归属用的，删掉会导致 Pages 取消验证，别动。

## 统计是怎么开的

Cloudflare Dashboard → **Analytics & Logs → Web Analytics** → 站点 `onlinechoice.online`
→ **Manage site** → Real User Measurements (RUM) 选 **Enable**
（"The JS Snippet will be automatically injected"）。

选 Enable 的前提是域名已经橙云代理。Cloudflare 会在边缘往每个 HTML 页面注入
beacon，上报端点是本域名下的 `/cdn-cgi/rum`。

> 注意：新域名**不会**自动出现在 Web Analytics 列表里，需要手动 Add a site 一次。
> 加完之后才是全自动，不用 token、不用改代码。

## 不要手动加 beacon

**不要**往 HTML 里贴 `static.cloudflareinsights.com/beacon.min.js` 的片段。
边缘已经注入了，再贴一份会变成两个 beacon，可能重复计数。

这一点对上游 master 仓库的页面模板同样适用 —— 本仓库的页面是生成出来的，
但统计不依赖生成产物，所以模板里也什么都不用加。

## 在哪里看数据

**Analytics & Logs → Web Analytics**（账户层级，不在域名菜单里）：

- 访问量（page views）、独立访问（visits）
- 热门页面、来源站点（referrers）
- 国家/地区、设备、浏览器、操作系统
- Core Web Vitals（LCP / INP / CLS）

数据有几分钟延迟。

## 两套统计的区别

| | Web Analytics | 域名 → Analytics（zone 层） |
|---|---|---|
| 原理 | 浏览器 JS beacon | 边缘服务端计数 |
| 会被广告拦截器挡 | 会 | 不会 |
| 包含爬虫/机器人 | 默认过滤 | 全部计入 |
| 适合看 | 真人访问行为 | 总流量、带宽、缓存命中、错误率 |

两边数字对不上是正常的。看真人用前者，看流量用后者。

## 怎么验证注入还在工作

1. 浏览器打开站点，Ctrl+U 看源码，`</body>` 前应有一个
   `<script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/…"`
   并带 `integrity` 属性 —— 这两个特征说明是边缘注入的，不是手写的
2. F12 → Network → 过滤 `rum` → 刷新，应有一条 `/cdn-cgi/rum` 请求返回 **204**

如果第 1 步搜不到，先确认记录还是橙云；如果第 2 步被 blocked，是本机广告拦截插件，
换无痕窗口测。

## 可选：更细的自建统计

想统计「某一道题被点开多少次」这类业务指标，Web Analytics 做不到。
需要另做 Cloudflare Worker + KV/D1：页面 `fetch()` 上报事件，Worker 写 KV，
再做一个受 Cloudflare Access 保护的后台页读取。目前未实现。
