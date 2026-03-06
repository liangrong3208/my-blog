---
layout: home
title: 首页
---

欢迎来到我的技术博客！这里记录我的技术学习、项目实践和一些思考。

## 最新文章

<ul>
  {% for post in site.posts limit:5 %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    </li>
  {% endfor %}
</ul>

[查看所有文章](/archive/)
