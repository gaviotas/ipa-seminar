---
layout: home
title: ""
---

<section class="hero-card">
  <p class="eyebrow">IPA Seminar Project</p>
  <h1>How to Take a Memorable Picture?</h1>
  <p class="subtitle">Empowering Users with Actionable Feedback (CVPR 2026)</p>
  <p>
    MemCoach 프로젝트 페이지 흐름을 따라, 논문 핵심 논리와 시각 자료를 발표 친화적으로 재구성했습니다.
  </p>
  <p class="hero-actions">
    <a class="btn-primary" href="{{ site.baseurl }}{% post_url 2026-04-14-how-to-take-a-memorable-picture-memcoach %}">논문 포스트 보기</a>
    <a class="btn-secondary" href="{{ '/presentation-notes/' | relative_url }}">발표 노트 보기</a>
  </p>
  <div class="link-row">
    <a class="link-chip" href="https://arxiv.org/abs/2602.21877" target="_blank" rel="noopener">Paper</a>
    <a class="link-chip" href="https://laitifranz.github.io/MemCoach/" target="_blank" rel="noopener">Project Page</a>
    <a class="link-chip" href="https://github.com/laitifranz/MemCoach" target="_blank" rel="noopener">Code</a>
    <a class="link-chip" href="https://huggingface.co/datasets/laitifranz/MemBench" target="_blank" rel="noopener">MemBench</a>
  </div>
</section>

<section class="section-card">
  <h2>Key Contributions</h2>
  <div class="grid-3">
    <article class="info-card">
      <h3>MemFeed</h3>
      <p>기억도 향상을 위한 actionable feedback 생성 태스크를 명시적으로 정의.</p>
    </article>
    <article class="info-card">
      <h3>MemCoach</h3>
      <p>Training-free teacher-student steering으로 실용적 피드백 생성.</p>
    </article>
    <article class="info-card">
      <h3>MemBench</h3>
      <p>비교 가능한 평가를 위한 벤치마크 데이터셋과 파이프라인 공개.</p>
    </article>
  </div>
</section>

<section class="section-card">
  <h2>Abstract Snapshot</h2>
  <blockquote>
    핵심 전환: “얼마나 기억에 남는가를 맞추는 문제”에서 “더 기억에 남도록 어떻게 찍을지 안내하는 문제”로 이동.
  </blockquote>
</section>

<section class="section-card">
  <h2>Method</h2>
  <figure class="media-panel">
    <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/method.webp" alt="MemCoach method overview" loading="lazy" />
    <figcaption>Teacher-Student contrastive generation과 activation steering을 결합한 MemCoach 파이프라인.</figcaption>
  </figure>
</section>

<section class="section-card">
  <h2>MemBench</h2>
  <div class="media-grid">
    <figure class="media-panel">
      <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/data-pipeline.webp" alt="MemBench data pipeline" loading="lazy" />
      <figcaption>MemBench 생성 및 평가 파이프라인.</figcaption>
    </figure>
    <figure class="media-panel">
      <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/statistics.webp" alt="MemBench statistics" loading="lazy" />
      <figcaption>MemBench 통계 및 분포 요약.</figcaption>
    </figure>
  </div>
</section>

<section class="section-card">
  <h2>Results</h2>
  <div class="media-grid">
    <figure class="media-panel">
      <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/qualitatives-feedback.webp" alt="MemCoach qualitative results" loading="lazy" />
      <figcaption>Qualitative feedback 예시.</figcaption>
    </figure>
    <figure class="media-panel">
      <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/table-results.webp" alt="MemCoach quantitative table" loading="lazy" />
      <figcaption>Quantitative 결과 비교 테이블.</figcaption>
    </figure>
  </div>
</section>

<section class="section-card">
  <h2>Demo</h2>
  <div class="video-frame">
    <video controls muted loop playsinline preload="metadata" poster="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/teaser/assets/photo_before.jpg">
      <source src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/teaser/assets/kling_blend_frames.mp4" type="video/mp4" />
    </video>
  </div>
  <div class="media-grid">
    <figure class="media-panel">
      <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/teaser/assets/photo_before.jpg" alt="Before applying feedback" loading="lazy" />
      <figcaption>Before</figcaption>
    </figure>
    <figure class="media-panel">
      <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/teaser/assets/photo_after.jpg" alt="After applying feedback" loading="lazy" />
      <figcaption>After</figcaption>
    </figure>
  </div>
</section>

## Quick Access
- [논문 리뷰 포스트]({{ site.baseurl }}{% post_url 2026-04-14-how-to-take-a-memorable-picture-memcoach %})
- [5분/10분 발표 노트]({{ '/presentation-notes/' | relative_url }})
- [참고 링크 모음]({{ '/resources/' | relative_url }})
