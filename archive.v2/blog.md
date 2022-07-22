---
layout: page
title: Blog
permalink: /blog/
---

<!-- - This section belongs to the Papers/Journals that I've read or will read. Topics aren't specified. Everything will be mixed, but it's more like a personal collection - <strong><a href="{{ '/papers' | prepend: site.baseurl | prepend: site.url }}">Papers</a></strong> -->

<!-- - I write things and take notes related to my studies here on <strong><a href="{{ '/study' | prepend: site.baseurl | prepend: site.url }}">Educate!</a></strong> -->

- My safe space and quoteblog - __[Damn Right, Internet](https://damnrightinternet.tumblr.com/)__

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
