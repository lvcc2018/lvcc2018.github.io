---
title: Blog
layout: default
---

# 技术博客

大模型算法、Agent 架构、量化策略的技术笔记与思考。

---

{% for post in site.posts %}
### {{ post.date | date: "%Y-%m-%d" }} — [{{ post.title }}]({{ post.url }})
> {{ post.summary }}

{% endfor %}
