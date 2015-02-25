---
layout: archives
title: Blog Posts
---

{% for category in site.categories %}
  <h2><a href="{{ site.baseurl}}/categories/{{ category | first }}">{{ category | first }}</a></h2>
  <div class="post-list">
  {% for posts in category %}
    {% for post in posts %}
      {% if post.title %}
  <div class="post-list-title"><a href="{{ post.url }}">{{ post.title }}</a></div>
  <div class="post-list-date">{{ post.date | date_to_string }}</div>
      {% endif %}
    {% endfor %}
  {% endfor %}
  </div>
{% endfor %}
