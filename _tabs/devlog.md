---
layout: page
icon: fas fa-pen-nib
title: Devlog
order: 2
permalink: /devlog/
---

{% assign devlog_posts = site.posts
  | where_exp: "p", "p.categories contains 'devlog'"
  | sort: "date"
  | reverse
%}

<div class="mz-cards mz-cards--devlog">
  {% for p in devlog_posts %}
    <a class="mz-card-link" href="{{ p.url | relative_url }}">
      <article class="mz-card">

        {% if p.project %}
          <div class="mz-card-project">{{ p.project }}</div>
        {% endif %}

        <time class="mz-card-date">
          {{ p.date | date: "%Y-%m-%d" }}
        </time>

        <h3 class="mz-card-title">{{ p.title }}</h3>

        {% if p.summary %}
          <p class="mz-card-summary">{{ p.summary }}</p>
        {% else %}
          <p class="mz-card-summary">{{ p.excerpt | strip_html | truncate: 120 }}</p>
        {% endif %}

        {% if p.tech %}
        <div class="mz-card-tags">
          {% for t in p.tech %}
            <span class="mz-card-tag">{{ t }}</span>
          {% endfor %}
        </div>
        {% endif %}

      </article>
    </a>

{% endfor %}

</div>
