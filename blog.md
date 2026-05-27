---
title: Blog
layout: default
---

# Blog

LLM training, Agent architecture, and model alignment.

{% for post in site.posts %}
<article class="blog-card">
  <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
  <div class="blog-card-meta">
    <time>{{ post.date | date: "%B %d, %Y" }}</time>
    <span class="blog-card-tags">{% for tag in post.tags %}<code>{{ tag }}</code> {% endfor %}</span>
  </div>
  {% if post.summary %}
  <p class="blog-card-summary">{{ post.summary }}</p>
  {% endif %}
  <a class="blog-card-link" href="{{ post.url }}">Read →</a>
</article>
{% endfor %}

{% if site.posts.size == 0 %}
<p><em>No posts yet.</em></p>
{% endif %}
