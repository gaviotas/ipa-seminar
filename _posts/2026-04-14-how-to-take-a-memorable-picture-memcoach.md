---
layout: post
title: "How to Take a Memorable Picture?"
subtitle: "Empowering Users with Actionable Feedback (CVPR 2026)"
date: 2026-04-14 20:00:00 +0900
categories: [paper-review, computer-vision]
tags: [memorability, mllm, cvpr2026, memcoach, membench]
description: "논문 원문 없이도 세미나 전달이 가능하도록 MemFeed 정식화, MemCoach 알고리즘, 실험 설계와 결과를 통합 정리한 기술 리뷰"
image: /assets/images/og-memcoach.svg
---

> 목표: 이 포스트만으로 세미나 청중이 논문 핵심 아이디어, 알고리즘, 실험 결과를 이해할 수 있게 구성

## Executive Summary
이 논문은 image memorability를 "점수 예측"에서 "사용자 행동 지시" 문제로 전환합니다.

핵심 제안:
1. **MemFeed**: memorability 개선을 위한 actionable feedback 생성 문제 정의
2. **MemCoach**: training-free activation steering으로 MLLM 피드백 분포 이동
3. **MemBench**: 데이터셋 + 평가 프로토콜(IR/RM/PPL) 제안

핵심 메시지:
- 기존 질문: "이 사진이 얼마나 기억에 남는가?"
- 새로운 질문: "더 기억에 남게 만들려면 무엇을 바꿔야 하는가?"

---

## 1) Background and Motivation
기존 memorability 연구는 크게 두 축이었습니다.
- **Prediction**: 이미지에서 scalar memorability score 회귀
- **Editing**: 이미지 자체를 수정해 memorability 증가 유도

문제:
- prediction은 actionable하지 않음
- editing은 사용자의 촬영 의사결정 과정을 설명하지 못함

논문은 capture-time support 관점에서 문제를 재정의합니다.
즉, 사람에게 "무엇을 바꾸라"고 말할 수 있는 모델이 필요합니다.

---

## 2) Task Definition: Memorability Feedback (MemFeed)
논문의 정식화(Section 3.1):
- source image `x_S`
- destination image `x_D`
- memorability predictor `M(·)`
- `m_S = M(x_S), m_D = M(x_D)`

목표:
- 피드백 `a`를 생성해서 사용자 수정 후 `m_D > m_S`를 만족하도록 유도

요구 조건:
1. **Actionable**: 실행 가능한 행동 단위 지시
2. **Interpretable**: 사람에게 읽히는 자연어
3. **Memorability-oriented**: aesthetic 일반론이 아니라 기억도 향상 중심

---

## 3) Why Baseline MLLMs Fail (Paper Evidence)
논문은 먼저 zero-shot MLLM 한계를 보여줍니다.

- LaMem yes/no 질의 기반 memorability 판별에서 Spearman이 거의 0 수준
  - QWEN2.5VL: -0.06
  - INTERNVL3.5: -0.01
  - IDEFICS3: -0.07
  - LLAVA-OV: 0.08
- inter-annotator upper bound: 0.68

또한 zero-shot feedback은 editing baseline 대비 IR 이득이 제한적입니다.

결론:
- 일반 MLLM은 memorability-specific direction이 내재적으로 약함
- 추가 학습 또는 steering 메커니즘이 필요

---

## 4) MemCoach Method (Technical Deep Dive)
MemCoach는 **teacher-student steering** 기반 3단계 구성입니다.

### 4.1 Contrasting Data Generation
씬 `i`의 이미지 집합 `X_i = {x_1^i, ..., x_M^i}`에서:
- `x_S^i`: least memorable
- `x_D^i`: most memorable

Teacher `ϕ_teach`가 privileged feedback 생성:
- `f_i^+ = ϕ_teach(x_S^i, x_D^i, p_a)`

Student `ϕ_stud`가 neutral feedback 생성:
- `f_i^- = ϕ_stud(x_S^i, p_m)`

대조 데이터:
- `F^+ = {f_i^+}_i`
- `F^- = {f_i^-}_i`

Prompt 의미:
- `p_a`: source→destination 변환에 필요한 행동 추출
- `p_m`: source 이미지의 memorability를 높이기 위한 행동 제안

### 4.2 Steering Vector Extraction
동일 `(x_S^i, p_m)` 조건에서 assistant 답변만 `f_i^+`/`f_i^-`로 바꾸어 student activation 수집.

레이어 `l`에서 steering vector:

```text
r^(l) = (1/N) * Σ_i ( h_{+,i}^(l) - h_{-,i}^(l) )
```

직관:
- `h_+ - h_-`는 "memorability-aware direction"
- 평균 벡터 `r^(l)`는 그 방향의 distilled latent signal

### 4.3 Inference-Time Steering
추론 시 student activation 이동:

```text
h_tilde^(l) = h^(l) + α * r^(l)
```

- `α`: steering strength
- 논문 기본값: `l=12`, `α=55` (InternVL3.5-8B)

장점:
- training-free
- model-agnostic (activation 접근 가능 시)
- low-data에서도 적용 가능

<figure class="media-panel">
  <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/method.webp" alt="MemCoach method overview" loading="lazy" />
  <figcaption>MemCoach pipeline (paper figure).</figcaption>
</figure>

---

## 5) MemBench: Dataset and Evaluation
MemBench는 PPR10K 기반으로 구축된 memorability-feedback 벤치마크입니다.

논문 보고 통계:
- 약 10K images
- 1,570 scenes
- scene당 평균 6.5 images

구축 파이프라인:
1. scene grouping
2. predictor `M`로 memorability scoring
3. scene 내부 ranking
4. low→high image pair 생성
5. captioning model로 actionable feedback 생성

<figure class="media-panel">
  <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/data-pipeline.webp" alt="MemBench pipeline" loading="lazy" />
  <figcaption>MemBench data generation and evaluation protocol.</figcaption>
</figure>

### 5.1 Evaluation Metrics
평가 축은 2개입니다.

1. **Editing-based effectiveness**
- 편집 모델 `e(x_S, a)`로 edited image `x_hat_D` 생성
- **IR**: `P[ m_D >= m_S ]`
- **RM**: `(m_D - m_S) / m_S`

2. **Feedback likelihood**
- memorability-aware GT feedback에 대한 perplexity
- 낮을수록 alignment가 좋음

강점:
- "말이 그럴듯한지"와 "실제로 좋아지는지"를 분리 측정

---

## 6) Main Experimental Results
아래 표는 논문 reported 수치를 그대로 요약한 것입니다.

### 6.1 Table 2 Summary (MemFeed Benchmarks)
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th>Family</th>
        <th>Model</th>
        <th>IR ↑</th>
        <th>RM% ↑</th>
        <th>PPL ↓</th>
      </tr>
    </thead>
    <tbody>
      <tr><td>Editing baseline</td><td>Empty feedback</td><td>0.68</td><td>3.72</td><td>n.d.</td></tr>
      <tr><td>Teacher oracle</td><td>LLAVA-OV</td><td>0.74</td><td>5.93</td><td>5.73</td></tr>
      <tr><td>Teacher oracle</td><td>IDEFICS3</td><td>0.80</td><td>9.84</td><td>29.21</td></tr>
      <tr><td>Teacher oracle</td><td>QWEN2.5VL</td><td>0.83</td><td>10.16</td><td>2.34</td></tr>
      <tr><td>Teacher oracle</td><td>INTERNVL3.5</td><td>0.85</td><td>11.92</td><td>2.40</td></tr>
      <tr><td>Aesthetic specialized</td><td>AESEXPERT</td><td>0.73</td><td>6.67</td><td>5.97</td></tr>
      <tr><td>Aesthetic specialized</td><td>Q-INSTRUCT</td><td>0.73</td><td>5.31</td><td>5.36</td></tr>
      <tr><td>Zero-shot</td><td>GPT-5 mini</td><td>0.75</td><td>7.03</td><td>n.d.</td></tr>
      <tr><td>Zero-shot</td><td>LLAVA-OV</td><td>0.70</td><td>5.87</td><td>7.58</td></tr>
      <tr><td>Zero-shot</td><td>IDEFICS3</td><td>0.73</td><td>6.64</td><td>20.19</td></tr>
      <tr><td>Zero-shot</td><td>QWEN2.5VL</td><td>0.68</td><td>4.26</td><td>10.23</td></tr>
      <tr><td>Zero-shot</td><td>INTERNVL3.5</td><td>0.73</td><td>5.47</td><td>5.49</td></tr>
      <tr><td><strong>Ours</strong></td><td><strong>MemCoach (InternVL)</strong></td><td><strong>0.80</strong></td><td><strong>7.21</strong></td><td><strong>4.99</strong></td></tr>
    </tbody>
  </table>
</div>

포인트:
- InternVL zero-shot→MemCoach에서 IR/RM/PPL 동시 개선
- training-free임에도 경쟁력 있는 수치 달성

### 6.2 Table 3 Summary (Cross-backbone Generalization)
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th>Backbone</th>
        <th>Baseline IR</th>
        <th>MemCoach IR</th>
        <th>ΔIR</th>
      </tr>
    </thead>
    <tbody>
      <tr><td>LLAVA-OV</td><td>0.70</td><td>0.73</td><td>+4.29%</td></tr>
      <tr><td>IDEFICS3</td><td>0.73</td><td>0.75</td><td>+2.74%</td></tr>
      <tr><td>QWEN2.5VL</td><td>0.68</td><td>0.74</td><td>+8.82%</td></tr>
      <tr><td>INTERNVL3.5</td><td>0.73</td><td>0.80</td><td>+9.59%</td></tr>
    </tbody>
  </table>
</div>

포인트:
- 특정 모델 트릭이 아니라 steering 방향의 일반성 시사

---

## 7) Ablation and Mechanistic Reading
논문에서 중요한 ablation 메시지:

1. **Data efficiency**
- low-data에서 steering이 LoRA fine-tuning 대비 효율적인 구간 존재

2. **Steering coefficient α**
- α 증가 초기에 성능 증가, 이후 saturation

3. **Difference operator design**
- sample-wise difference 후 평균(제안식)이
- 평균 후 difference(`diff(mean)`)보다 성능이 좋음

해석:
- memorability signal이 per-sample contrast에서 더 선명하게 추출됨

---

## 8) Critical Appraisal
강점:
1. 문제 설정의 실용성(사용자 행동 단위)
2. training-free 적용성
3. 텍스트 품질+실효성 분리 평가

한계:
1. 평가가 `predictor M`과 `editor e`에 의존
2. memorability의 개인/문화 편차를 평균화
3. distinctiveness를 잃는 방향의 과도한 정규화 실패 가능

열린 질문:
- 개인화 memorability profile을 넣으면 성능이 어떻게 바뀌는가?
- 편집 모델 의존성을 줄인 human-in-the-loop 평가는 어떻게 설계할 것인가?
- 단기 recall과 장기 recall 간 상관을 어떻게 검증할 것인가?

---

## 9) Reproducibility Checklist (for Lab Discussion)
1. 동일 scene 기반 pair 구성 재현 여부
2. teacher/student prompt consistency 검증
3. steering layer `l`, coefficient `α` 스윕 로그 확보
4. predictor/editor 바꿨을 때 ranking robustness 확인
5. IR/RM/PPL 외 human evaluation 보강

---

## 10) Seminar Closing Message
이 논문의 공헌은 성능 수치 하나가 아니라, 목표 함수를 바꿨다는 점입니다.

- Memorability는 더 이상 "맞히는 값"만이 아니라,
- 사람에게 "가르칠 수 있는 행동 지식"으로 다뤄질 수 있습니다.

## References
- Paper (arXiv): [https://arxiv.org/abs/2602.21877](https://arxiv.org/abs/2602.21877)
- Project page: [https://laitifranz.github.io/MemCoach/](https://laitifranz.github.io/MemCoach/)
- Code: [https://github.com/laitifranz/MemCoach](https://github.com/laitifranz/MemCoach)
- MemBench dataset: [https://huggingface.co/datasets/laitifranz/MemBench](https://huggingface.co/datasets/laitifranz/MemBench)
