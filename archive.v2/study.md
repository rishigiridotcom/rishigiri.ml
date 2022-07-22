---
layout: page
title: Study Blog
permalink: /study/
---

<section class="post-list">
  <div class="container">
    <ul>
      {% for post in site.notes %}
        <li> <a href="{{ post.url }}">{{ post.title }}</a> </li>
      {% endfor %}
    </ul>
  </div>
</section>
