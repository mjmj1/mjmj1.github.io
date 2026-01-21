---
layout: page
icon: fas fa-book
title: Study
order: 1
permalink: /study/
---

<link rel="stylesheet" href="{{ '/assets/css/cards.css?v=20260121' | relative_url }}">

{% assign study_posts = site.posts
  | where_exp: "p", "p.categories contains 'study'"
  | sort: "date"
  | reverse
%}

<div class="cards">
  {% for p in study_posts %}
    <a class="card-link" href="{{ p.url | relative_url }}">
      <div class="card">

        <h3>{{ p.title }}</h3>
        <p class="meta">{{ p.date | date: "%Y-%m-%d" }}</p>

        <div class="tags">
          {% if p.tech %}
            {% for t in p.tech %}
              <span class="tag">{{ t }}</span>
            {% endfor %}
          {% endif %}
        </div>

      </div>
    </a>

{% endfor %}

</div>
