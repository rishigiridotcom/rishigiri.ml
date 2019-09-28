---
layout: page
title: Blog
permalink: /blog/
---

<br>
- I write things and take notes related to my studies here on __[riblog!](https://riblog.ml)__

- For Technical Posts, click __[here!][wordpress-blog]__

<section class="post-list">
  <div class="container">
    {% for post in site.posts %}
      {% unless post.next %}
        <h2 class="category-title">{{ post.date | date: '%Y' }}</h2>
      {% else %}
        {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
        {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
        {% if year != nyear %}
          <h2 class="category-title">{{ post.date | date: '%Y' }}</h2>
        {% endif %}
      {% endunless %}
      <article class="post-item">
        <span class="post-meta date-label">{{ post.date | date: "%b %d" }}</span>
        <li>
          <div class="article-title">
            <a class="post-link" href="{{ post.url | prepend: site.baseurl | prepend: site.url }}">{{ post.title }}</a> <span class="post-update">{{ post.update }}</span>
          </div>
        </li>
      </article>
    {% endfor %}
  </div>

</section>

[wordpress-blog]: https://rishicodes.wordpress.com
[damnrightinternet]: https://damnrightinternet.tumbrl.com
