---
layout: single
author_profile: true
---

  {% for post in site.posts %}
      <a href="{{ post.url }}">{{ post.title }}</a>
  {% endfor %}