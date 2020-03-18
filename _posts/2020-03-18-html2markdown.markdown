---
layout:       post
title:        "【网络开发】如何把要想保存的文章转为 Markdown 格式"
subtitle:     "如何把要想保存的文章转为 Markdown 格式"
date:         2020-03-18 14:41:00
author:       "ZhangPY"
header-img:   "img/post-bg-os-metro.jpg"
header-mask:  0.3
catalog:      true
multilingual: true
tags:
    - 网络开发
---

<!-- Chinese Version -->
<div class="zh post-container">
    {% capture about_zh %}{% include posts/2020-03-18-html2markdown.md %}{% endcapture %}
    {{ about_zh | markdownify }}
</div>

<!-- English Version -->
<div class="en post-container">
    {% capture about_en %}{% include posts/en.md %}{% endcapture %}
    {{ about_en | markdownify }}
</div>
