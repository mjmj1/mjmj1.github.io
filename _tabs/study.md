---
layout: page
icon: fas fa-book
title: Study
order: 1
permalink: /study/
---

<link rel="stylesheet" href="{{ '/assets/css/cards.css' | relative_url }}">

{% assign study_posts = site.posts | where_exp: "p", "p.categories contains 'study'" | sort: "date" | reverse %}

<div class="cards">
{% for p in study_posts %}
  <div class="card">
    <h3><a href="{{ p.url | relative_url }}">{{ p.title }}</a></h3>
    <p class="meta">{{ p.date | date: "%Y-%m-%d" }}</p>

    {% if p.summary %}
      <p class="desc">{{ p.summary }}</p>
    {% elsif p.excerpt %}
      <p class="desc">{{ p.excerpt | strip_html | strip_newlines | truncate: 140 }}</p>
    {% endif %}

    {% if p.tags %}
      <div class="tags">
        {% for t in p.tags limit: 6 %}
          <span class="tag">{{ t }}</span>
        {% endfor %}
      </div>
    {% endif %}

    <div class="actions">
      <a class="btn primary" href="{{ p.url | relative_url }}">Read</a>
    </div>

  </div>
{% endfor %}
</div>
