---
layout: default
title: Blog
permalink: /blog/
---
<h2>Latest Posts</h2>

<ul>
  {% for post in site.posts %}
    <li>
      <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    </li>
  {% endfor %}
</ul>