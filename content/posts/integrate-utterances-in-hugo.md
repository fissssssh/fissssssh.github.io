---
title: "在 Hugo 中集成 utterances 评论组件"
date: 2023-01-24T02:22:36+08:00
draft: false
tags:
  - Hugo
---

utterances 是基于 GitHub issues 构建的轻量级评论小部件，通过此组件可以将 Github issues 应用于任何网站

## 前置条件

- 一个公开的 github repo
- 在该 repo 中安装[utterances app](https://github.com/apps/utterances)

## 简单集成

1. 打开[utterances](https://utteranc.es/)官网
2. 在 configuration 节点填写 repo 信息等，填写好后会生成对应的组件代码
3. 我使用的是 [PaperMod 主题][1]，将生成好的代码放入`layouts/partials/comments.html`

   ```html
   <!--layouts/partials/comments.html-->

   <script
     src="https://utteranc.es/client.js"
     repo="[ENTER REPO HERE]"
     issue-term="pathname"
     theme="github-light"
     label="Comment"
     crossorigin="anonymous"
     async
   ></script>
   ```

   然后在`config.yaml`中开启评论功能即可简单集成

   ```yaml
   params:
     comments: true
   ```

## 动态主题适配

[PaperMod 主题][1]有明亮和暗黑两种模式，而 comments 组件的主题是编码时就写在`script`标签中的，为了使 comments 组件的主题适配 [PaperMod 主题][1]，有以下两个步骤：

### 主题动态初始化

```html
<!-- layouts/partials/comments.html -->

<script>
  (function () {
    const theme =
      localStorage.getItem("pref-theme") === "dark"
        ? "github-dark"
        : "github-light";
    const comment = document.createElement("script");
    comment.src = "https://utteranc.es/client.js";
    comment.crossorigin = "anonymous";
    comment.async = true;
    comment.setAttribute("repo", "[ENTER REPO HERE]");
    comment.setAttribute("issue-term", "pathname");
    comment.setAttribute("label", "Comment");
    comment.setAttribute("theme", theme);
    document.body.appendChild(comment);
  })();
</script>
```

### 主题动态切换

```html
<!-- layouts/partials/extend_footer.html -->

{{- if (not site.Params.disableThemeToggle) }}
<script>
  document.getElementById("theme-toggle").addEventListener("click", () => {
    const iframe = document.querySelector(".utterances-frame");
    const message = {
      type: "set-theme",
      theme:
        localStorage.getItem("pref-theme") === "light" // if current theme is light, then set comment widget's theme to dark, otherwise light.
          ? "github-dark"
          : "github-light",
    };
    iframe?.contentWindow.postMessage(message, "https://utteranc.es");
  });
</script>
{{- end }}
```

> [PaperMod 主题][1]会将当前主题存在 `localStorage`->`pref-theme` 下

## 参考

- [Adaptive dark theme](https://github.com/utterance/utterances/issues/299#issuecomment-626125665)
- [Dynamic theme changing](https://github.com/utterance/utterances/issues/549#issuecomment-907606127)

[1]: https://github.com/adityatelange/hugo-PaperMod
