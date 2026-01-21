---
layout: page
icon: fas fa-pen-nib
title: Devlog
order: 2
permalink: /devlog/
---

<link rel="stylesheet" href="{{ '/assets/css/cards.css?v=20260121' | relative_url }}">

{% assign devlog_posts = site.posts
  | where_exp: "p", "p.categories contains 'devlog'"
  | sort: "date"
  | reverse
%}

<div class="cards devlog-cards">
  {% for p in devlog_posts %}
    <a class="card-link" href="{{ p.url | relative_url }}">
      <div class="card">

        {% if p.project %}
          <div class="project-label">{{ p.project }}</div>
        {% endif %}

        <h3>{{ p.title }}</h3>
        <p class="meta">{{ p.date | date: "%Y-%m-%d" }}</p>

        <div class="tags">
          {% if p.tech and p.tech.size > 0 %}
            {% for t in p.tech %}
              <span class="tag">{{ t }}</span>
            {% endfor %}
          </div>
        {% endif %}

      </div>

{% endfor %}

</div>
