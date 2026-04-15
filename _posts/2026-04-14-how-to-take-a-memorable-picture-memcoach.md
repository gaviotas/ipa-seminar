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

## Paper At A Glance
<div class="badge-row">
  <span class="paper-badge paper-badge-highlight">CVPR 2026 Highlight</span>
</div>

- **Title**: *How to Take a Memorable Picture? Empowering Users with Actionable Feedback*
- **Venue**: CVPR 2026
- **Recognition**: Highlight Award
- **arXiv**: 2602.21877 (v2, 2026-03-19)
- **Project**: [MemCoach](https://laitifranz.github.io/MemCoach/)

**Authors**
- Francesco Laiti<sup>1,2,3</sup>
- Davide Talon<sup>4</sup>
- Jacopo Staiano<sup>1</sup>
- Elisa Ricci<sup>1,4</sup>

**Affiliations**
- <sup>1</sup> University of Trento
- <sup>2</sup> University of Pisa
- <sup>3</sup> Travelbrain srl
- <sup>4</sup> Fondazione Bruno Kessler

## Project Teaser (Camera Frame)
<div class="video-frame">
  <iframe src="https://laitifranz.github.io/MemCoach/static/images/teaser/teaser_animation.html" title="MemCoach teaser animation with camera frame" loading="lazy" allowfullscreen></iframe>
</div>

## Executive Summary
이 논문은 image memorability를 "점수 예측"에서 "사용자 행동 지시" 문제로 전환합니다.

핵심 제안:
1. **MemFeed**: memorability 개선을 위한 actionable feedback 생성 문제 정의
2. **MemCoach**: training-free activation steering으로 MLLM 피드백 분포 이동
3. **MemBench**: 데이터셋 + 평가 프로토콜(IR/RM/PPL) 제안

핵심 메시지:
- 기존 질문: "이 사진이 얼마나 기억에 남는가?"
- 새로운 질문: "더 기억에 남게 만들려면 무엇을 바꿔야 하는가?"

### 이 페이지를 읽는 순서 (세미나용)
1. `2) Task Definition`에서 문제를 고정
2. `4) MemCoach Method`에서 식 2개로 알고리즘 이해
3. `6) Main Experimental Results`에서 수치와 효과 크기 확인
4. `8) Critical Appraisal`로 토론 포인트 정리

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

## Related Work Map
MemCoach는 기존 연구의 정확한 한 축에만 속하기보다, 여러 흐름의 교차점에 위치합니다.

### A. Memorability Prediction
- *What Makes an Image Memorable?* (CVPR 2011)
- *Understanding and Predicting Image Memorability at a Large Scale* (ICCV 2015)

이 계열은 "이미지가 얼마나 기억에 남는가"를 예측하는 데 집중합니다.
MemCoach는 여기서 한 걸음 더 나아가, **예측값을 행동 지시로 변환**하는 문제를 다룹니다.

### B. Memorability Editing / Modification
- *How to Make an Image More Memorable?* (2017)
- face/image memorability modification 계열 연구

이 계열은 모델이 이미지를 직접 수정해 memorability를 높이려 합니다.
반면 MemCoach는 이미지를 대신 바꾸지 않고, **사람이 직접 적용할 수 있는 feedback**을 생성합니다.

### C. Human Guidance for Photography
- *Adaptive Photographic Composition Guidance* (CHI 2020)
- *The Photographer’s Eye* (CVPR 2025)

이 계열은 사용자의 촬영/구도 판단을 돕는다는 점에서 MemCoach와 가장 실용적으로 가깝습니다.
다만 대부분 memorability 자체를 직접 target으로 삼지는 않습니다.

### D. Feedback / Critique Generation
- aesthetics critique / image feedback generation 계열
- 예: VILA (CVPR 2023), 사진 비평형 MLLM 연구들

이 계열은 자연어 피드백을 생성하지만, 보통 목표는 aesthetic quality 또는 general critique입니다.
MemCoach는 여기서 목적 함수를 **future recall**로 특화합니다.

### E. Training-Free Steering / Activation Engineering
- Style Vectors for Steering Generative LLMs
- CAST (conditional activation steering)
- 기타 activation engineering 계열

이 계열은 행동/스타일/응답 성향을 weight update 없이 steering하는 연구입니다.
MemCoach의 방법론적 핵심은 여기에 속하지만, 응용 대상이 **actionable feedback for memorability**라는 점이 차별점입니다.

정리하면 MemCoach의 위치는 다음과 같습니다.
- 문제 측면: memorability prediction → actionable feedback으로 확장
- 방법 측면: training-free activation steering 채택
- 응용 측면: capture-time human guidance에 초점

---

## 2) Task Definition: Memorability Feedback (MemFeed)
논문의 정식화(Section 3.1):
- source image \\(x_S\\)
- destination image \\(x_D\\)
- memorability predictor \\(M(\cdot)\\)
- \\(m_S = M(x_S),\ m_D = M(x_D)\\)

목표:
- 피드백 \\(a\\)를 생성해서 사용자 수정 후 \\(m_D > m_S\\)를 만족하도록 유도

중요한 해석:
- 여기서 \\(x_D\\)는 실제 inference 시 모델 입력으로 주어지는 이미지가 아니다.
- \\(x_D\\)는 benchmark 구성과 teacher supervision을 위해 정의되는 target view이며, student는 추론 시 \\(x_S\\)만 관찰한다.

요구 조건:
1. **Actionable**: 실행 가능한 행동 단위 지시
2. **Interpretable**: 사람에게 읽히는 자연어
3. **Memorability-oriented**: aesthetic 일반론이 아니라 기억도 향상 중심

### Notation Quick Table
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th>Symbol</th>
        <th>Meaning</th>
      </tr>
    </thead>
    <tbody>
      <tr><td>\(x_S\)</td><td>Source image (현재 사용자 입력 이미지)</td></tr>
      <tr><td>\(x_D\)</td><td>Destination image / target view (benchmark·teacher supervision용)</td></tr>
      <tr><td>\(M(\cdot)\)</td><td>Memorability predictor</td></tr>
      <tr><td>\(m_S, m_D\)</td><td>각 이미지의 memorability score</td></tr>
      <tr><td>\(a\)</td><td>생성할 actionable feedback 텍스트</td></tr>
      <tr><td>\(\phi_{\mathrm{teach}}, \phi_{\mathrm{stud}}\)</td><td>Teacher / Student MLLM</td></tr>
      <tr><td>\(r^{(l)}\)</td><td>Layer \(l\) 의 memorability steering vector</td></tr>
      <tr><td>\(\alpha\)</td><td>Steering strength coefficient</td></tr>
    </tbody>
  </table>
</div>

---

## 3) Why Baseline MLLMs Fail (Paper Evidence)
논문은 먼저 zero-shot MLLM 한계를 보여줍니다.

- **LaMem**은 대규모 image memorability benchmark로, 각 이미지에 대해 사람 실험 기반 memorability score가 부여된 데이터셋이다.
- 논문은 LaMem 이미지 하나를 MLLM에 넣고 `"Is this image memorable? Output only yes or no."`라고 질문한 뒤,
  `yes` 토큰 likelihood와 `no` 토큰 likelihood의 상대값을 memorability score proxy로 사용한다.
- 이렇게 얻은 모델별 ranking을 LaMem의 ground-truth memorability ranking과 **Spearman rank correlation**으로 비교하면 거의 0 수준이다.
  - QWEN2.5VL: -0.06
  - INTERNVL3.5: -0.01
  - IDEFICS3: -0.07
  - LLAVA-OV: 0.08
- inter-annotator upper bound: 0.68
  - 이는 서로 다른 사람 집단끼리의 memorability 판단 일치도 상한선에 해당하며, 인간 수준 ceiling처럼 해석할 수 있다.

또한 zero-shot feedback은 **editing baseline** 대비 IR 이득이 제한적입니다.
- 여기서 **IR (Improvement Ratio)** 는 편집 후 이미지의 memorability가 원본보다 커진 비율을 뜻한다.
- 여기서 editing baseline은 feedback으로 **빈 문자열(empty string)** 을 넣은 경우다.
- 즉, memorability-aware instruction 없이 편집 모델만 동작시켰을 때를 최소 기준선으로 두고 비교한 것이다.

결론:
- 일반 MLLM은 memorability-specific direction이 내재적으로 약함
- 추가 학습 또는 steering 메커니즘이 필요

---

## 4) MemCoach Method (Technical Deep Dive)
MemCoach는 **teacher-student steering** 기반 3단계 구성입니다.

### 4.1 Contrasting Data Generation
씬 \\(i\\)의 이미지 집합 \\(X_i = \{x_1^i, \ldots, x_M^i\}\\)에서:
- \\(x_S^i\\): least memorable
- \\(x_D^i\\): most memorable

Teacher \\(\phi_{\text{teach}}\\)가 privileged feedback 생성:
- \\(f_i^+ = \phi_{\text{teach}}(x_S^i, x_D^i, p_a)\\)

Student \\(\phi_{\text{stud}}\\)가 neutral feedback 생성:
- \\(f_i^- = \phi_{\text{stud}}(x_S^i, p_m)\\)

대조 데이터:
- \\(F^+ = \{f_i^+\}_i\\)
- \\(F^- = \{f_i^-\}_i\\)

Prompt 의미:
- `p_a`: source→destination 변환에 필요한 행동 추출
- `p_m`: source 이미지의 memorability를 높이기 위한 행동 제안

코드 기준 실제 prompt 요약:
- `p_a`: "Image A를 Image B로 바꾸기 위해 필요한 actions를 생성하라."  
  추가 제약: 이미지 자체를 직접 지칭하지 말고, 각 action은 self-contained해야 하며, JSON `actions` 리스트로 출력
- `p_m`: "입력 이미지의 memorability를 높이기 위해 필요한 actions를 생성하라."  
  추가 제약: precise and well-informative semantic actions를 출력하고, 설정에 따라 comma-separated text 또는 JSON `actions` 리스트 형태 사용

### 4.2 Steering Vector Extraction
동일 \\((x_S^i, p_m)\\) 조건에서 assistant 답변만 \\(f_i^+\\)/\\(f_i^-\\)로 바꾸어 student activation 수집.

여기서 핵심은 **입력 조건을 통제한 controlled contrast**라는 점이다.
- 바뀌지 않는 것: source image \\(x_S^i\\), user prompt \\(p_m\\)
- 바뀌는 것: assistant answer only

즉 student는 같은 이미지와 같은 질문을 보지만,
- 한 번은 teacher가 만든 memorability-aware feedback \\(f_i^+\\)
- 한 번은 student가 원래 내놓는 neutral feedback \\(f_i^-\\)

을 assistant 슬롯에 둔 두 개의 시퀀스를 각각 forward pass 한다.

이렇게 하면 activation 차이를 거의 순수하게 **response style / memorability-awareness 차이**로 해석할 수 있다.
즉, 같은 입력에 대해 neutral response에서 memorability-aware response로 갈 때 latent representation이 어느 방향으로 이동하는지를 측정하는 절차라고 볼 수 있다.

레이어 `l`에서 steering vector:

$$
r^{(l)} = \frac{1}{N}\sum_i\left(h_{+,i}^{(l)} - h_{-,i}^{(l)}\right)
$$

직관:
- \\(h_+ - h_-\\)는 memorability-aware direction
- 평균 벡터 \\(r^{(l)}\\)는 그 방향의 distilled latent signal

### 4.3 Inference-Time Steering
추론 시 student activation 이동:

$$
\tilde{h}^{(l)} = h^{(l)} + \alpha\, r^{(l)}
$$

- \\(\alpha\\): steering strength
- 논문 기본값: \\(l=12,\ \alpha=55\\) (InternVL3.5-8B)

장점:
- training-free
- model-agnostic (activation 접근 가능 시)
- low-data에서도 적용 가능

### 4.4 Algorithm Sketch (Pseudocode)
<div class="pseudo-code">
  <p><strong>Input:</strong> scene-grouped images \(X_i\), predictor \(M\), teacher \(\phi_{\text{teach}}\), student \(\phi_{\text{stud}}\)</p>
  <p>1) For each scene \(i\):</p>
  <p>&nbsp;&nbsp;- pick \(x_S^i = \arg\min_x M(x)\), \(x_D^i = \arg\max_x M(x)\)</p>
  <p>&nbsp;&nbsp;- generate \(f_i^+ = \phi_{\text{teach}}(x_S^i, x_D^i, p_a)\)</p>
  <p>&nbsp;&nbsp;- generate \(f_i^- = \phi_{\text{stud}}(x_S^i, p_m)\)</p>
  <p>2) Collect student activations \(h_{+,i}^{(l)}, h_{-,i}^{(l)}\) and compute</p>
  <p>&nbsp;&nbsp;\(r^{(l)} = \frac{1}{N}\sum_i\left(h_{+,i}^{(l)} - h_{-,i}^{(l)}\right)\)</p>
  <p>3) At inference, steer activation:</p>
  <p>&nbsp;&nbsp;\(\tilde{h}^{(l)} = h^{(l)} + \alpha r^{(l)}\)</p>
  <p>4) Decode steered student output as final memorability feedback</p>
</div>

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
- 편집 모델 \\(e(x_S, a)\\)로 edited image \\(\hat{x}_D\\) 생성
- **IR**: \\(P[m_D \ge m_S]\\)
- **RM**: \\((m_D - m_S) / m_S\\)

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
        <th>Model</th>
        <th>IR ↑</th>
        <th>RM% ↑</th>
        <th>PPL ↓</th>
      </tr>
    </thead>
    <tbody>
      <tr class="table-group"><td colspan="4">Editing baseline</td></tr>
      <tr><td>Empty feedback</td><td>0.68</td><td>3.72</td><td>n.d.</td></tr>

      <tr class="table-group"><td colspan="4">Teacher oracles</td></tr>
      <tr><td>LLAVA-OV</td><td>0.74</td><td>5.93</td><td>5.73</td></tr>
      <tr><td>IDEFICS3</td><td>0.80</td><td>9.84</td><td>29.21</td></tr>
      <tr><td>QWEN2.5VL</td><td>0.83</td><td>10.16</td><td>2.34</td></tr>
      <tr><td>INTERNVL3.5</td><td>0.85</td><td>11.92</td><td>2.40</td></tr>

      <tr class="table-group"><td colspan="4">Aesthetics specialized</td></tr>
      <tr><td>AESEXPERT</td><td>0.73</td><td>6.67</td><td>5.97</td></tr>
      <tr><td>Q-INSTRUCT</td><td>0.73</td><td>5.31</td><td>5.36</td></tr>

      <tr class="table-group"><td colspan="4">Zero-shot baselines</td></tr>
      <tr><td>GPT-5 mini</td><td>0.75</td><td>7.03</td><td>n.d.</td></tr>
      <tr><td>LLAVA-OV</td><td>0.70</td><td>5.87</td><td>7.58</td></tr>
      <tr><td>IDEFICS3</td><td>0.73</td><td>6.64</td><td>20.19</td></tr>
      <tr><td>QWEN2.5VL</td><td>0.68</td><td>4.26</td><td>10.23</td></tr>
      <tr><td>INTERNVL3.5</td><td>0.73</td><td>5.47</td><td>5.49</td></tr>

      <tr class="table-group"><td colspan="4">MemCoach (Ours)</td></tr>
      <tr><td><strong>MemCoach (InternVL)</strong></td><td><strong>0.80</strong></td><td><strong>7.21</strong></td><td><strong>4.99</strong></td></tr>
    </tbody>
  </table>
</div>

포인트:
- InternVL zero-shot→MemCoach에서 IR/RM/PPL 동시 개선
- training-free임에도 경쟁력 있는 수치 달성

효과 크기(InternVL zero-shot 대비):
- IR: `0.73 → 0.80` (absolute `+0.07`, relative `+9.59%`)
- RM%: `5.47 → 7.21` (absolute `+1.74`, relative `+31.81%`)
- PPL: `5.49 → 4.99` (absolute `-0.50`, lower is better)

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

### 6.3 Figure 7: Qualitative Feedback from MemCoach
<figure class="media-panel">
  <img src="https://raw.githubusercontent.com/laitifranz/MemCoach/main/docs/static/images/paper/qualitatives-feedback.webp" alt="Figure 7 qualitative feedback from MemCoach" loading="lazy" />
  <figcaption>Figure 7. Qualitative feedback from MemCoach. Source 이미지(좌)에서 자연어 피드백을 적용해 destination 이미지(우)로 이동하는 사례를 보여주며, RM 변화(성공/실패 케이스)를 함께 제시합니다.</figcaption>
</figure>

해석 포인트:
1. 제안 문장이 단순 설명이 아니라 실행 가능한 동작(자세/시선/구도/가림요소 제거)으로 구성됨
2. positive case에서는 RM 상승이 뚜렷하게 나타남
3. failure case는 \"distinctiveness\"를 제거하는 방향(예: out-of-context 요소 제거)에서 발생 가능

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

### Threats to Validity (세미나 토론용)
1. **Metric proxy risk**: IR/RM이 predictor `M` 품질에 직접 영향받음
2. **Editor coupling risk**: 편집 모델 `e`의 성향이 결과 수치를 좌우할 수 있음
3. **Distribution risk**: PPR10K 기반 분포와 실제 사용자 촬영 분포 간 괴리 가능
4. **Preference gap**: memorability 개선이 개인 미학 선호와 충돌할 가능성

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
