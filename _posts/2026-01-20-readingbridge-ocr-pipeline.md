---
layout: post
title: "PDF → OCR 파이프라인 설계"
date: 2026-01-20
categories: [project, devlog]
project: Reading Bridge
summary: "PDF 렌더링부터 OCR 추출, 전처리까지의 서버 파이프라인을 분리해 구성한 흐름을 정리."
pin: true
---

요약 문단.

<!--more-->

## 흐름

- PDF → 이미지 변환
- OCR 추출
- 텍스트 전처리
