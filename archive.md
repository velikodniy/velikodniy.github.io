---
layout: page
title: Архив
---

<ul>
  {% assign posts = site.posts | sort: 'date' | reverse %}
  {% for post in posts %}
  <li>
  {{ post.date | date: "%Y-%m-%d" }} <a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
  </li>
  {% endfor %}
</ul>

