---
layout: home
title: ""
---

<section class="hero-card">
  <p class="eyebrow">IPA Seminar · Technical Review</p>
  <h1>How to Take a Memorable Picture?</h1>
  <p class="subtitle">Empowering Users with Actionable Feedback (CVPR 2026)</p>
  <p>
    이 페이지는 논문 비전문 청중도 따라올 수 있도록 문제 정의를 명확히 설명하면서,
    AI 박사 청중을 위해 수식/평가설계/ablation까지 포함한 기술 세미나 구조로 구성했습니다.
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
  <h2>Seminar Flow</h2>
  <ol>
    <li>문제 전환: prediction에서 actionable feedback으로</li>
    <li>MemFeed 정식화: `m_D > m_S`를 만족하는 피드백 생성</li>
    <li>MemCoach 알고리즘: contrasting pairs, steering vector, inference-time shift</li>
    <li>MemBench 평가: IR/RM/PPL의 의미와 해석</li>
    <li>ablation과 일반화: data efficiency, alpha saturation, cross-backbone behavior</li>
    <li>한계와 연구 방향: evaluator bias, personalization, human study</li>
  </ol>
</section>

<section class="section-card">
  <h2>Key Technical Points</h2>
  <ul>
    <li>Steering vector: r^(l) = (1/N) * Σ_i (h_{+,i}^(l) - h_{-,i}^(l))</li>
    <li>Inference steering: h_tilde^(l) = h^(l) + α * r^(l)</li>
    <li>논문 기본 세팅: InternVL3.5-8B, l=12, α=55</li>
  </ul>
</section>

<section class="section-card">
  <h2>Read First</h2>
  <p>
    그림 위주 요약이 아니라, 방법론과 실험 설계를 중심으로 재작성한 기술 리뷰는 아래 포스트에서 확인할 수 있습니다.
  </p>
  <p>
    <a class="btn-primary" href="{{ site.baseurl }}{% post_url 2026-04-14-how-to-take-a-memorable-picture-memcoach %}">Technical Deep Dive 열기</a>
  </p>
</section>
