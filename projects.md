---
layout: page
title: Projects
permalink: /projects/
---

# My Projects

{% for project in site.projects %}
<div class="project-card">
  <h2><a href="{{ project.url }}">{{ project.title }}</a></h2>
  <p>{{ project.description }}</p>
  <p><strong>Technologies:</strong> {{ project.technologies }}</p>
  <a href="{{ project.github_link }}" class="button">View on GitHub</a>
</div>
{% endfor %}
