---
layout: page
title: Articles
tags: [ python, django ]
---

##### Categories

{% for article in site.articles %}
  <h5>
    <a href="{{ article.url }}">
      {{ article.title }}
    </a>
  </h5>
  <p>{{ article.short_content | markdownify }}</p>
{% endfor %}
{% for article in page.tags %}
  <span>{{ article }}</span>
{% endfor %}