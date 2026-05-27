---
title: Blog
layout: default
---

# 技术博客

LLM training, Agent architecture, and model alignment.

<div class="post-list-home">
{% for post in site.posts %}
<a href="{{ post.url }}" class="post-item">
  <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
  <span class="post-title">{{ post.title }}</span>
  <span class="post-tags">{% for tag in post.tags %}<code>{{ tag }}</code> {% endfor %}</span>
</a>
{% endfor %}
</div>
