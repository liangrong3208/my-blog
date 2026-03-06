---
layout: page
title: 所有文章
permalink: /archive/
---

## 文章列表

<ul class="post-list">
  {% for post in site.posts %}
    <li>
      <span class="post-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
      <h3>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </h3>
      {% if post.categories.size > 0 %}
        <span class="post-categories">{{ post.categories | join: ", " }}</span>
      {% endif %}
    </li>
  {% endfor %}
</ul>
