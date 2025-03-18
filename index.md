---
layout: home
title: Home
---

# Hello, I'm Ishan Kakodkar

I'm a [your profession] specializing in [your skills]. Welcome to my portfolio and blog.

## Latest Projects

{% for project in site.projects limit:3 %}
- [{{ project.title }}]({{ project.url }})
{% endfor %}

## Recent Posts

{% for post in site.posts limit:3 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
