---
layout: page
icon: fas fa-briefcase
title: Portfolio
order: 3
permalink: /portfolio/
---

# 🙇 김만종 | Game & Backend Developer

AI NPC와 멀티플레이 네트워크 구조를 중심으로 게임 시스템을 설계·구현한 경험이 있습니다.  
Unity 기반 분산 권한 멀티플레이 구조와 ML-Agents를 활용한 AI NPC 설계 경험을 바탕으로, 대표 프로젝트 **The Zoo**를 통해 **inD 게임 브릿지 데이 인디게임 공모전 우수상**과 **교내 캡스톤디자인 전시회 장려상**을 수상했습니다.

<sub>관심 분야: Unity · Netcode for GameObjects · ML-Agents · Game Backend</sub>

## 🏢 이력

| 기간                    | 이름                   | 내용                                                                                                |
| ----------------------- | ---------------------- | --------------------------------------------------------------------------------------------------- |
| 2023.05.22 ~ 2024.03.01 | 비엔에프테크놀로지(주) | 원자력 발전소 모니터링·조작 SW 개발, 운영 SW 유지보수, <br/>테스트 자동화 매크로, SRS/SDD 문서 작성 |
| 2025.06.30 ~ 2025.08.20 | 아이와즈 (인턴)        | AI 기반 엣지 디바이스 얼굴 인식<br/>Jetson Nano, ONNX/TensorRT 파이프라인                           |

## 🛠 기술 스택

<div class="stack-grid">

  <div class="stack-card">
    <div class="stack-title">Game Engine</div>
    <div class="stack-icons">
      <img class="stack-icon"
           src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/unity/unity-original.svg"
           alt="Unity">
    </div>
    <div class="stack-desc">멀티플레이 게임 개발 및 AI NPC 구현</div>
  </div>

  <div class="stack-card">
    <div class="stack-title">Web / Backend</div>
    <div class="stack-icons">
      <img class="stack-icon" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/spring/spring-original.svg" alt="Spring">
      <img class="stack-icon" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/django/django-plain.svg" alt="Django">
      <img class="stack-icon" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/postgresql/postgresql-original.svg" alt="PostgreSQL">
    </div>
    <div class="stack-desc">서비스 API, OCR·AI 파이프라인 구현</div>
  </div>

  <div class="stack-card">
    <div class="stack-title">Tools</div>
    <div class="stack-icons">
      <img class="stack-icon" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/postman/postman-original.svg" alt="Postman">
      <img class="stack-icon" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/github/github-original.svg" alt="GitHub">
      <img class="stack-icon" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/docker/docker-original.svg" alt="Docker">
    </div>
  </div>

</div>

{%- assign project_posts = site.posts
  | where_exp: "p", "p.categories contains 'devlog'"
  | where_exp: "p", "p.project"
  | sort: "date"
  | reverse
-%}

{%- assign featured_posts = project_posts
  | where_exp: "p", "p.feature == true"
-%}

{%- assign normal_posts = project_posts
  | where_exp: "p", "p.feature != true"
-%}

## Featured Projects

<div class="mz-feature-cards">
  {% for p in featured_posts %}
    <article class="mz-feature-card">

      <div class="mz-feature-top">
        <div class="mz-feature-kicker">FEATURED</div>
        <time class="mz-feature-date">{{ p.date | date: "%Y-%m-%d" }}</time>
      </div>

      <h3 class="mz-feature-title">
        {% if p.project %}{{ p.project }}{% else %}{{ p.title }}{% endif %}
      </h3>

      {% if p.summary %}
        <p class="mz-feature-summary">{{ p.summary }}</p>
      {% endif %}

      {% if p.awards %}
        <div class="mz-feature-awards">
          {% for a in p.awards %}
            <span class="mz-feature-award">{{ a }}</span>
          {% endfor %}
        </div>
      {% endif %}

      {% if p.overview or p.tech %}
        <div class="mz-feature-overview-box">
        {% if p.overview %}
          <dl class="mz-feature-overview">
            {% if p.overview.period %}<div><dt>기간</dt><dd>{{ p.overview.period }}</dd></div>{% endif %}
            {% if p.overview.team %}<div><dt>인원</dt><dd>{{ p.overview.team }}</dd></div>{% endif %}
            {% if p.overview.role %}<div><dt>역할</dt><dd>{{ p.overview.role }}</dd></div>{% endif %}
            {% if p.overview.platform %}<div><dt>플랫폼</dt><dd>{{ p.overview.platform }}</dd></div>{% endif %}
            {% if p.overview.link %}<div><dt>링크</dt><dd><a href="{{ p.overview.link }}">{{ p.overview.link }}</a></dd></div>{% endif %}
          </dl>
        {% endif %}

        {% if p.tech %}
          <div class="mz-feature-tags">
            {% for t in p.tech %}
              <span class="mz-feature-tag">{{ t }}</span>
            {% endfor %}
          </div>
        {% endif %}
        </div>
      {% endif %}

      {%- assign snippet = "" -%}

      {%- assign parts = p.content | split: "<!--pf-start-->" -%}
      {%- if parts.size > 1 -%}
        {%- assign snippet = parts[1] | split: "<!--pf-end-->" | first -%}
      {%- else -%}
        {%- assign parts2 = p.content | split: "<!--pf-a-start-->" -%}
        {%- if parts2.size > 1 -%}
          {%- assign snippet = parts2[1] | split: "<!--pf-a-end-->" | first -%}
        {%- endif -%}
      {%- endif -%}

      <div class="mz-feature-content">
        {% if snippet != "" %}
          {{ snippet }}
        {% endif %}

      </div>

      <div class="mz-feature-actions">
        <a class="mz-feature-btn" href="{{ p.url | relative_url }}">글 전체 보기</a>
        <a class="mz-feature-btn mz-feature-btn--primary"
           href="{{ p.url | relative_url }}#impl">핵심 로직 바로가기</a>
      </div>

    </article>

{% endfor %}

</div>

## Projects

<div class="mz-cards mz-cards--portfolio">
  {% for p in normal_posts %}
    <a class="mz-card-link" href="{{ p.url | relative_url }}">
      <article class="mz-card">
        {% if p.project %}
          <div class="mz-card-project">{{ p.project }}</div>
        {% endif %}

        <time class="mz-card-date">{{ p.date | date: "%Y-%m-%d" }}</time>
        <h3 class="mz-card-title">{{ p.title }}</h3>

        {% if p.summary %}
          <p class="mz-card-summary">{{ p.summary }}</p>
        {% else %}
          <p class="mz-card-summary">{{ p.excerpt | strip_html | truncate: 140 }}</p>
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
