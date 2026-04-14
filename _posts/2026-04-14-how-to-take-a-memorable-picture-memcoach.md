---
layout: post
title: "How to Take a Memorable Picture?"
subtitle: "Empowering Users with Actionable Feedback (CVPR 2026)"
date: 2026-04-14 20:00:00 +0900
categories: [paper-review, computer-vision]
tags: [memorability, mllm, cvpr2026, memcoach, membench]
description: "Memorability Feedback(MemFeed)와 MemCoach의 기술적 핵심(정식화, steering, 평가 프로토콜)을 다루는 세미나용 리뷰"
image: /assets/images/og-memcoach.svg
---

> 대상 청중: AI/CV 전공 박사 연구자

## TL;DR
이 논문은 memorability 연구를 **예측(regression)**에서 **행동 유도(actionable feedback)**로 전환합니다.
핵심은 training-free activation steering으로 MLLM의 피드백 생성 분포를 이동시키는 점입니다.

## 1. Problem Setting (MemFeed)
입력 이미지 `x_S`에 대해, 자연어 피드백 `a`를 생성하여 사용자가 촬영/구도/표정을 바꿨을 때 더 높은 기억도 이미지 `x_D`로 이동하도록 하는 문제입니다.

논문의 정식화(Section 3.1):
- `m_S = M(x_S)`, `m_D = M(x_D)`
- 목표: `m_D > m_S`
- 여기서 `M`은 memorability predictor

이 문제는 score prediction과 다릅니다.
- prediction: 현재 상태 측정
- MemFeed: 개선 방향 제시

## 2. Why Zero-shot MLLM Is Not Enough
논문은 먼저 MLLM의 memorability 이해 한계를 보여줍니다.
- LaMem 기준 yes/no memorability 질의에서 Spearman 상관이 거의 0 수준
- zero-shot feedback은 editing baseline 대비 IR 개선이 작음

즉, 일반 MLLM의 “시각-언어 능력”만으로는 memorability-aware guidance를 안정적으로 만들기 어렵다는 전제가 성립합니다.

## 3. MemCoach: Technical Core
MemCoach는 3단계 파이프라인입니다.

### 3.1 Contrasting Data Generation
씬 `i`의 이미지 집합 `X_i`에서
- `x_S^i`: least memorable image
- `x_D^i`: most memorable image

Teacher 모델 `ϕ_teach`로 privileged feedback 생성:
- `f_i^+ = ϕ_teach(x_S^i, x_D^i, p_a)`

Student 모델 `ϕ_stud`의 neutral feedback 생성:
- `f_i^- = ϕ_stud(x_S^i, p_m)`

대조쌍:
- `F^+ = {f_i^+}_i`
- `F^- = {f_i^-}_i`

### 3.2 Steering Vector Extraction
같은 `(x_S^i, p_m)` 조건에서 assistant 답변만 `f_i^+` vs `f_i^-`로 바꿔 student activation을 수집합니다.

레이어 `l`에서 steering vector:

```text
r^(l) = (1/N) * Σ_i ( h_{+,i}^(l) - h_{-,i}^(l) )
```

핵심은 sample-wise difference를 먼저 계산하고 평균내는 구조입니다.
논문 ablation에서도 이 방식이 `diff(mean)`보다 좋다고 보고합니다.

### 3.3 Inference-Time Steering
추론 시 student activation `h^(l)`를 아래처럼 이동:

```text
h_tilde^(l) = h^(l) + α * r^(l)
```

- `α`: steering strength
- 논문 기본값: `l=12`, `α=55` (InternVL3.5-8B 기준)

장점:
- training-free
- backbone-agnostic (activation 접근 가능하면 적용 가능)

<figure class="media-panel">
  <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/method.webp" alt="MemCoach method overview" loading="lazy" />
  <figcaption>MemCoach 방법 개요. Source: official project/repo figure.</figcaption>
</figure>

## 4. MemBench: Dataset and Evaluation Design
MemBench는 PPR10K 기반으로 구축됩니다.
- 약 10K 이미지
- 1,570 scenes
- scene당 평균 6.5 이미지

생성 파이프라인:
1. scene grouping
2. memorability regression (`M`)
3. scene 내부 ranking
4. low→high pair로 action feedback 생성

### 4.1 Metrics
논문은 평가를 2축으로 설계합니다.

1. Editing-based effectiveness
- 편집 모델 `e(x_S, a)`로 feedback 적용 결과 `x_hat_D` 생성
- **IR (Improvement Ratio)**: `m_D >= m_S` 비율
- **RM (Relative Memorability)**: `(m_D - m_S) / m_S`

2. Feedback likelihood
- ground-truth memorability-aware feedback에 대한 perplexity

이 설계는 “문장이 자연스러운가”와 “실제로 개선되는가”를 동시에 보려는 점이 강점입니다.

<figure class="media-panel">
  <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/data-pipeline.webp" alt="MemBench pipeline" loading="lazy" />
  <figcaption>MemBench 구축 및 평가 파이프라인. Source: official project/repo figure.</figcaption>
</figure>

## 5. Quantitative Results (Paper)
논문 Table 2(InternVL 기반 MemCoach) 핵심 수치:

| Model | IR ↑ | RM% ↑ | PPL ↓ |
|---|---:|---:|---:|
| Zero-shot InternVL3.5 | 0.73 | 5.47 | 5.49 |
| Zero-shot GPT-5 mini | 0.75 | 7.03 | n.d. |
| MemCoach (InternVL) | **0.80** | **7.21** | **4.99** |

해석:
- zero-shot 대비 IR/RM/PPL 동시 개선
- aesthetics-specialized MLLM들 대비도 경쟁력 있음

Table 3 일반화 결과도 중요합니다.
- LLAVA, IDEFICS, QWEN, INTERNVL 백본 전반에서 IR 개선 보고
- 즉, 특정 백본 특화 기법이 아니라 activation-space steering의 일반성이 메시지

## 6. Ablation Insights
논문에서 세미나 토론에 유용한 포인트:
- **Data efficiency**: low-data에서도 LoRA fine-tune 대비 유리한 구간 보고
- **Steering coefficient α**: 증가 초기에 성능 상승, 이후 포화
- **Teacher choice**: teacher를 바꾸면 성능 변화가 생기지만 steering 이득 자체는 유지

## 7. Critical Discussion
강점:
1. 문제 정의 자체가 명확하고 실사용 맥락과 정합적
2. training-free로 구현 부담이 낮음
3. 평가를 텍스트 품질 + 편집 기반 효과로 분리해 설계

한계:
1. `M` predictor와 `e` editor에 대한 의존성(평가 파이프라인 편향 가능)
2. memorability의 개인/문화 편차를 평균화해 다룸
3. “distinctiveness”를 과도하게 정규화하는 failure mode 가능

## 8. Seminar Takeaways
1. MemFeed는 memorability 연구의 objective를 바꾼다.
2. MemCoach의 본질은 parameter update가 아니라 latent steering이다.
3. 향후 확장은 personalization(사용자별 기억 특성)과 closed-loop human study가 핵심이다.

## References
- Paper (arXiv): [https://arxiv.org/abs/2602.21877](https://arxiv.org/abs/2602.21877)
- Project page: [https://laitifranz.github.io/MemCoach/](https://laitifranz.github.io/MemCoach/)
- Code: [https://github.com/laitifranz/MemCoach](https://github.com/laitifranz/MemCoach)
- MemBench dataset: [https://huggingface.co/datasets/laitifranz/MemBench](https://huggingface.co/datasets/laitifranz/MemBench)
