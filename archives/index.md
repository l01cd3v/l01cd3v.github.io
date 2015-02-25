---
layout: archives
title: Blog Posts
---

<div class="post-list">
{% for post in site.posts %}
  <div class="post-list-title"><a href="{{ post.url }}">{{ post.title }}</a></div>
  <div class="post-list-date">{{ post.date | date_to_string }}</div>
{% endfor %}
</div>
