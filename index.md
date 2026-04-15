---
layout: home
title: ""
---

<section class="hero-card">
  <p class="eyebrow">IPA Seminar · Technical Review</p>
  <h1>IPA Seminar Paper Notes</h1>
  <p class="subtitle">원문을 따로 열지 않아도 세미나 발표가 가능하도록 정리한 기술 리뷰 포스트 모음</p>
  <p>
    현재는 computer vision 쪽의 <em>MemCoach</em>와 NLP 쪽의 <em>CoGen</em>을
    정리해두었고, 앞으로 같은 형식의 세미나용 포스트를 계속 추가할 수 있도록 구성했습니다.
  </p>
  <p class="hero-actions">
    <a class="btn-primary" href="{{ '/papers/' | relative_url }}">포스트 모아보기</a>
    <a class="btn-secondary" href="{{ '/presentation-notes/' | relative_url }}">발표 노트 보기</a>
  </p>
  <div class="link-row">
    <a class="link-chip" href="https://arxiv.org/abs/2602.21877" target="_blank" rel="noopener">Paper</a>
    <a class="link-chip" href="https://laitifranz.github.io/MemCoach/" target="_blank" rel="noopener">Project Page</a>
    <a class="link-chip" href="https://github.com/laitifranz/MemCoach" target="_blank" rel="noopener">Code</a>
    <a class="link-chip" href="https://huggingface.co/datasets/laitifranz/MemBench" target="_blank" rel="noopener">MemBench</a>
    <a class="link-chip" href="https://aclanthology.org/2024.emnlp-main.721/" target="_blank" rel="noopener">CoGen Paper</a>
    <a class="link-chip" href="https://github.com/lil-lab/cogen" target="_blank" rel="noopener">CoGen Code</a>
  </div>
</section>

<section class="section-card">
  <h2>Current Posts</h2>
  <div class="paper-card">
    <h3>How to Take a Memorable Picture?</h3>
    <p class="muted">CVPR 2026 · Memorability · MLLM Steering</p>
    <div class="badge-row">
      <span class="paper-badge paper-badge-highlight">Highlight</span>
    </div>
    <p>
      memorability prediction을 actionable feedback generation 문제로 바꾼 논문으로,
      MemFeed 정식화, MemCoach steering, MemBench 평가까지 정리했습니다.
    </p>
    <p><a class="btn-primary" href="{{ site.baseurl }}{% post_url 2026-04-14-how-to-take-a-memorable-picture-memcoach %}">MemCoach 포스트 보기</a></p>
  </div>
  <div class="paper-card">
    <h3>CoGen</h3>
    <p class="muted">EMNLP 2024 · Continual Learning · Coupled Speaker/Listener</p>
    <div class="badge-row">
      <span class="paper-badge paper-badge-award">Best Paper Award</span>
    </div>
    <p>
      reference game에서 generation과 comprehension을 함께 개선하는 방법으로,
      data sharing, joint inference, continual learning loop를 중심으로 정리했습니다.
    </p>
    <p><a class="btn-primary" href="{{ site.baseurl }}{% post_url 2026-04-15-cogen-learning-from-feedback %}">CoGen 포스트 보기</a></p>
  </div>
</section>

<section class="section-card">
  <h2>What These Posts Aim To Do</h2>
  <ul>
    <li>논문 메타데이터, 문제 설정, 방법론, 실험, 한계를 한 페이지 안에서 연결</li>
    <li>처음 나오는 benchmark, metric, baseline을 바로 설명해 원문 의존도를 낮춤</li>
    <li>AI 박사 청중을 위한 technical detail과 비전문 청중을 위한 맥락 설명을 함께 유지</li>
  </ul>
</section>

<section class="section-card">
  <h2>Read First</h2>
  <p>
    논문을 빠르게 훑고 싶다면 `Papers` 페이지에서 원하는 포스트를 열면 되고,
    발표 직전 점검용 요약은 `Presentation Notes`에서 확인할 수 있습니다.
  </p>
  <p>
    <a class="btn-primary" href="{{ '/papers/' | relative_url }}">Papers 열기</a>
  </p>
</section>
