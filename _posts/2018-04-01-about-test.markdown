---
layout:       post
title:        "test"
subtitle:     "test"
date:         2018-04-01 23:52:00
author:       "ZhangPY"
header-img:   "img/post-bg-os-metro.jpg"
header-mask:  0.3
catalog:      true
multilingual: true
tags:
    - test

---

<!-- Chinese Version -->
<div class="zh post-container">
    {% capture about_zh %}{% include posts/2018-04-01-test.md %}{% endcapture %}
    {{ about_zh | markdownify }}
</div>

<!-- English Version -->
<div class="en post-container">
    {% capture about_en %}{% include posts/en.md %}{% endcapture %}
    {{ about_en | markdownify }}
</div>