---
layout: page
title: Blog
permalink: /blog/
---

# Latest Posts

{% for post in site.posts %}
<div class="post-preview">
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
  <p>{{ post.excerpt }}</p>
  <a href="{{ post.url }}" class="read-more">Read More</a>
</div>
{% endfor %}
