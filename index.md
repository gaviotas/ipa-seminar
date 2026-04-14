---
layout: home
title: ""
---

<section class="hero-card">
  <p class="eyebrow">IPA Seminar Project</p>
  <h1>How to Take a Memorable Picture?</h1>
  <p class="subtitle">Empowering Users with Actionable Feedback (CVPR 2026)</p>
  <p>
    논문 요약을 넘어서 발표용 스토리라인, 핵심 메시지, 한계 토론 포인트까지
    한 번에 볼 수 있게 구성한 세미나 페이지입니다.
  </p>
  <p class="hero-actions">
    <a class="btn-primary" href="{{ site.baseurl }}{% post_url 2026-04-14-how-to-take-a-memorable-picture-memcoach %}">논문 포스트 보기</a>
    <a class="btn-secondary" href="{{ '/presentation-notes/' | relative_url }}">발표 노트 보기</a>
  </p>
</section>

<section class="grid-3">
  <article class="info-card">
    <h3>핵심 질문</h3>
    <p>"지금 이 장면에서 무엇을 바꾸면 더 기억에 남는 사진이 되는가?"</p>
  </article>
  <article class="info-card">
    <h3>핵심 기여</h3>
    <p>MemFeed 과제 정의, MemCoach 조향 방식, MemBench 벤치마크 공개.</p>
  </article>
  <article class="info-card">
    <h3>발표 초점</h3>
    <p>예측을 넘어 사용자 행동 변화를 유도하는 비전 시스템.</p>
  </article>
</section>

## Featured Visuals
<div class="media-grid">
  <figure class="media-panel">
    <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/teaser.webp" alt="MemCoach teaser figure" loading="lazy" />
    <figcaption>논문 teaser: 문제와 기여를 한 장으로 요약</figcaption>
  </figure>
  <figure class="media-panel">
    <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/method.webp" alt="MemCoach method figure" loading="lazy" />
    <figcaption>방법 개요: contrastive generation + steering inference</figcaption>
  </figure>
</div>

<div class="video-frame">
  <iframe src="https://www.youtube.com/embed/JkaxUblCGz0" title="MemCoach Demo Video" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</div>

## Quick Access
- [논문 리뷰 포스트]({{ site.baseurl }}{% post_url 2026-04-14-how-to-take-a-memorable-picture-memcoach %})
- [5분/10분 발표 노트]({{ '/presentation-notes/' | relative_url }})
- [참고 링크 모음]({{ '/resources/' | relative_url }})
