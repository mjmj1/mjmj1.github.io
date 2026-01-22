<%*
/**
 * Unified post template
 * - devlog / study content differs
 * - summary stored in front matter
 * - summary also rendered in body as "한 줄 요약"
 */

const category = await tp.system.suggester(
  ["Devlog", "Study"],
  ["devlog", "study"]
);

const slug = await tp.system.prompt("Post title (kebab-case)");
const today = tp.date.now("YYYY-MM-DD");

await tp.file.rename(`${today}-${slug}`);

const title = slug.replace(/-/g, " ");

const summary = await tp.system.prompt(
  "한 줄 요약 (optional)"
);

const project = await tp.system.prompt(
  "Project name? (optional)"
);

const techCsv = await tp.system.prompt(
  "Tech tags (comma separated, optional)"
);

const tech = (techCsv || "")
  .split(",")
  .map(s => s.trim())
  .filter(Boolean);
-%>
---
layout: post
title: "<% title %>"
date: "<% tp.date.now('YYYY-MM-DD HH:mm:ss') %> +0900"
categories: [<% category %>]
summary: "<% summary || '' %>"
project: "<% project || '' %>"
tech: [<% tech.join(', ') %>]
---
<%* if (category === "devlog") { %>

## 프로젝트 개요

> **한 줄 요약**  
> <% summary || "" %>
{: .prompt-tip }

- **기간**:
- **인원**:
- **역할**:
- **관련 링크**:

## 기술 선택 배경 / 기획 의도
- 
- 

## 책임 및 기여
- 
- 

## 결과

### 개선 방향 및 학습한 점

1. 
2. 
3. 

### 기술적 한계와 문제점

1. 
2. 
3. 

## 구현 상세 및 핵심 로직

### XXX.cs

##### **개요**

#### 로직

##### **핵심 코드**

```csharp
```

##### **설계 의도 / 코드 설명**

- 
-
-

---

<%* } else if (category === "study") { %>

## 학습 주제
- 무엇을 공부했는지
- 학습 계기

## 핵심 개념 정리
- 개념 1
- 개념 2
- 개념 3

## 코드 / 예제
```text
(필요 시 코드 또는 예제)
````

## 정리 및 해석

* 개념에 대한 이해
* 적용 시 주의점

## 개인 메모

* 헷갈렸던 부분
* 추가로 공부할 포인트

## 참고 자료

* 공식 문서
* 블로그 / 영상 링크

<%* } %>