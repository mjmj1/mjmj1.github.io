---
layout: page
icon: fas fa-briefcase
title: Portfolio
order: 3
permalink: /portfolio/
---

# 🙇 김만종 | Game & Backend Developer

AI NPC와 멀티플레이 네트워크 구조를 중심으로 게임 시스템을 설계·구현한 경험이 있습니다.  
Unity 기반 분산 권한 멀티플레이 구조와 ML-Agents를 활용한 AI NPC 설계 경험을 바탕으로, 대표 프로젝트 **The Zoo**를 통해 *inD 게임 브릿지 데이 인디게임 공모전 우수상*과 *교내 캡스톤디자인 전시회 장려상*을 수상했습니다.

<sub>관심 분야: Unity · Netcode for GameObjects · ML-Agents · Game Backend</sub>

## 🏢 이력

| 기간                    | 이름                   | 내용                                                                                              |
| ----------------------- | ---------------------- | ------------------------------------------------------------------------------------------------- |
| 2024.05.22 ~ 2025.03.01 | 비엔에프테크놀로지(주) | 원자력 발전소 모니터링·조작 SW 개발<br/>운영 SW 유지보수, 테스트 자동화 매크로, SRS/SDD 문서 작성 |
| 2025.06.30 ~ 2025.08.20 | 아이와즈 (인턴)        | AI 기반 엣지 디바이스 얼굴 인식<br/>Jetson Nano, ONNX/TensorRT 파이프라인                         |

## feature Projects

## Projects

{% assign project_posts = site.posts | where_exp: "p", "p.categories contains 'project'" %}

{% for proj in site.data.projects | sort: "order" %}

### {{ proj.title }}

{{ proj.one_liner }}

- 기간: {{ proj.period }}
- Tech: {{ proj.tech | join: " · " }}
  {% if proj.awards %}
- 성과: {{ proj.awards | join: " / " }}
  {% endif %}

{% assign posts_for_proj = project_posts | where: "project", proj.id %}

{% if posts_for_proj.size > 0 %}
**관련 글**
{% for p in posts_for_proj limit:5 %}

- [{{ p.title }}]({{ p.url | relative_url }}){% if p.summary %} — {{ p.summary }}{% endif %}
  {% endfor %}
  {% else %}
  관련 글 준비 중
  {% endif %}

---

{% endfor %}
