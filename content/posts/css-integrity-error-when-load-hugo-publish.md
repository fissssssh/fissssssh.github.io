---
title: "Css Integrity Error When Load Hugo Publish"
date: 2022-06-20T21:30:04+08:00
categories: misc
tags:
  - hugo
draft: false
---

## 问题描述

近日我使用 hugo 构建了我的博客，并通过 Github Action 将其发布在 Github Pages 上，刚开始还是很美好的，但是过一短时间以后打开页面发现样式全无，使用浏览器的开发者工具查看资源获取没有问题，但是在控制台却出现了这样一句话：

```plaintext
Failed to find a valid digest in the 'integrity' attribute for resource '***' with computed SHA-256 integrity '***'. The resource has been blocked.
```

## 寻找原因

我在 MDN 上寻找到了关于 [integrity](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link#attr-integrity) 的定义，大概描述就是这是一个签名，浏览器获取到相应资源后会用相同的方法计算一个签名，只有签名相同时才会加载对应的资源，如果两个签名不一致则是文件完整性被破坏（文件发生了改变）。

问题来了，整个发布过程是由 Github Action 全自动操作的，没有人为干预，文件为何会无缘无故改变呢？

答案是 Cloudflare。 Cloudflare 中默认会开启静态资源的缓存来提高网站的加载速度，可是为什么缓存会改变文件呢？缓存并不会改变文件，在 Cloudflare 的 `Speed` > `Optimization` 中有一个叫 Auto Minify 的选项，描述如下：

> Reduce the file size of source code on your website.
>
> **Note:** Purge cache to have your change take effect immediately.

这句话翻译过来就是 减小网站源代码文件体积的大小 ，同时该选项有 3 个子选项：

- `JavaScript`
- `CSS`
- `HTML`

默认都是选中状态，即默认会压缩我们的 js，css 和 html 文件。

## 解决问题

因为我们网站是 css 文件的签名验证出现了问题，因此我们取消 Auto Minify 下面的 `Css` 选项，然后在`Caching` > `Configuration` 中点击 Purge Cache 清除缓存，回到网站使用 <kbd>ctrl</kbd> + <kbd>F5</kbd> 强制刷新网页，问题解决。

## 其他解决方案

我使用的是 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题，可以将主题文件 `layouts/partials/head.html` 中 css 引用部分的 integrity 属性去除，同样可以解决，但我觉得主题毕竟是第三方库，在非必要情况下不要修改第三方库。

```diff
- <link crossorigin="anonymous" href="{{ $stylesheet.RelPermalink }}" integrity="{{ $stylesheet.Data.Integrity }}" rel="preload stylesheet" as="style">
+ <link crossorigin="anonymous" href="{{ $stylesheet.RelPermalink }}" rel="preload stylesheet" as="style">
```

## 结束语

遇到困难仔细分析，终会迎刃而解。
