---
title: Hexo 升级
date: 2019-01-24 17:22:19
categories:
- Hexo
tags:
- Hexo
---

## 转载：[将 Hexo 升级到 v3.5.0](https://tommy.net.cn/2018/02/26/upgrade-hexo-to-v3-5-0/)

其实这几个月以来，一直都想写点什么东西，很多时候笔记也做好了，但是拖延症一而再再而三的发作，进而导致到了毫无产出可言。

这不，托 Nison 同学的福，败家了一个新的 Macbook 笔记本。

新笔记本到了，最痛苦的就是各种配置环境和常用软件，以及回忆一大堆的账号密码了。

<!--more-->



当然少不了要同步一下我这万年不更新的博客了，结果一看，果然变化不小。

首先是，Node.js 的最新稳定版本已经变成了 [v8.9.4](https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V8.md#8.9.4) 啦！可是我还停留在 LTS v6 的年代啊。。。(╥╯^╰╥)

果断用 Brew 安装了 n 和最新 LTS 版本的 Node.js，开始了 Hexo 的升级之路。

其实所谓的升级，也很简单，先进入 Blog 的目录，看看到底有哪些需要更新的：

```
npm outdated
Package                              Current  Wanted  Latest  Location
hexo-deployer-git                      0.2.0   0.2.0   0.3.1  hexo-site
hexo-generator-search                  1.0.4   1.0.4   2.2.1  hexo-site
hexo-generator-seo-friendly-sitemap   0.0.19  0.0.19  0.0.21  hexo-site
hexo-renderer-ejs                      0.2.0   0.2.0   0.3.1  hexo-site
hexo-renderer-marked                  0.2.11  0.2.11   0.3.2  hexo-site
hexo-server                            0.2.2   0.2.2   0.3.1  hexo-site
```

嗯，还是有不少东西需要更新的，简单修改一下 `package.json` 文件：

```
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.5.0"
  },
  "dependencies": {
    "hexo": "^3.5.0",
    "hexo-deployer-git": "^0.3.1",
    "hexo-deployer-rsync": "^0.1.3",
    "hexo-excerpt": "^1.1.2",
    "hexo-generator-archive": "^0.1.5",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-feed": "^1.2.0",
    "hexo-generator-index": "^0.2.1",
    "hexo-generator-search": "^2.2.1",
    "hexo-generator-seo-friendly-sitemap": "0.0.21",
    "hexo-generator-sitemap": "^1.1.2",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.1",
    "hexo-renderer-marked": "^0.3.2",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-server": "^0.3.1"
  }
}
```

把 Hexo 的版本号从 `3.3.8` 修改为 `3.5.0`，其他的也根据情况更新一下。

都修改好了以后，就 npm 更新一下：

```
npm install --save
```

搞掂，运行 Hexo 看看效果：

```
$ hexo version
hexo: 3.5.0
hexo-cli: 1.0.4
os: Darwin 17.4.0 darwin x64
http_parser: 2.7.0
node: 8.9.4
v8: 6.1.534.50
uv: 1.15.0
zlib: 1.2.11
ares: 1.10.1-DEV
modules: 57
nghttp2: 1.25.0
openssl: 1.0.2n
icu: 59.1
unicode: 9.0
cldr: 31.0.1
tz: 2017b
```

你看到这篇文章就是 Hexo v3.5.0 发布的啦，不过 Hexo 官网好像完全没更新的样子，现在看到[最新的新闻](https://hexo.io/news/)还是介绍 v3.2.0 的。