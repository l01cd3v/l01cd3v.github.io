---
layout: archives
title: Blog Posts
---

{% for category in site.categories %}
  <h2><a href="{{ site.baseurl}}/categories/{{ category | first }}">{{ category | first }}</a></h2>
  {% for posts in category %}
    {% for post in posts %}
      {% if post.title %}
  {{ post.date | date_to_string }} &raquo; <a href="{{ post.url }}">{{ post.title }}</a>
      {% endif %}
    {% endfor %}
  {% endfor %}
{% endfor %}
