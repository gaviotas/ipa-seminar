---
layout: post
title: "CoGen"
subtitle: "Learning from Feedback with Coupled Comprehension and Generation (EMNLP 2024)"
date: 2026-04-15 21:10:00 +0900
categories: [paper-review, nlp]
tags: [emnlp2024, continual-learning, feedback-learning, reference-games, cogen]
description: "Reference game setting에서 comprehension과 generation을 결합해 continual learning을 수행하는 CoGen을, 문제 설정부터 joint inference와 data sharing까지 세미나용으로 정리한 기술 리뷰"
image: /assets/images/og-cogen.svg
---

> 목표: 이 포스트만으로 세미나 청중이 CoGen의 문제 설정, coupling 메커니즘, 학습식, 실험 결과를 이해할 수 있게 구성

## Paper At A Glance
<div class="badge-row">
  <span class="paper-badge paper-badge-award">EMNLP 2024 Best Paper Award</span>
</div>

- **Title**: *CoGen: Learning from Feedback with Coupled Comprehension and Generation*
- **Venue**: EMNLP 2024 Main Conference
- **Recognition**: Best Paper Award
- **Paper**: [ACL Anthology 2024.emnlp-main.721](https://aclanthology.org/2024.emnlp-main.721/)
- **arXiv**: [2408.15992](https://arxiv.org/abs/2408.15992)
- **Code**: [lil-lab/cogen](https://github.com/lil-lab/cogen)

**Authors**
- Mustafa Omer Gul
- Yoav Artzi

**Affiliation**
- Cornell University / Cornell Tech

## Executive Summary
이 논문은 language **generation**과 **comprehension**을 따로 학습하지 말고, 상호작용 안에서 서로를 도와가며 함께 개선하자는 제안입니다.

핵심 제안:
1. **Interaction setting**: speaker/listener reference game에서 사람과 상호작용하며 학습
2. **Training-time coupling**: 한 역할의 성공 사례를 다른 역할의 학습 데이터로 재사용하는 `data sharing`
3. **Inference-time coupling**: listener 분포와 speaker 분포를 결합하는 `joint inference`

핵심 메시지:
- generation이 잘되면 comprehension도 더 잘 학습할 수 있고
- comprehension이 좋아지면 generation도 더 pragmatic하게 변한다
- 이 coupling을 지속적 학습(continual learning) 안에서 돌리면 성능뿐 아니라 언어 자체도 인간 쪽으로 이동한다

### 이 페이지를 읽는 순서 (세미나용)
1. `2) Task Setup`에서 reference game을 먼저 고정
2. `4) CoGen Method`에서 `data sharing`과 `joint inference`를 구분
3. `6) Main Results`에서 성능 향상과 language trend를 같이 해석
4. `7) Critical Appraisal`로 RLHF와의 차이, 한계를 정리

---

## 1) Background and Motivation
대부분의 language-capable system은 comprehension과 generation을 별개 태스크처럼 다룹니다.

- **Comprehension**: 주어진 utterance를 해석해 정답을 고른다
- **Generation**: 주어진 목표를 표현하는 utterance를 만든다

논문은 이 분리가 비효율적이라고 봅니다.

직관은 단순합니다.
- 좋은 speaker는 listener가 어떤 신호를 잘 해석하는지 알아야 한다
- 좋은 listener는 speaker가 어떤 표현을 사용할지를 어느 정도 예측해야 한다

즉, 두 능력은 분리된 모듈이 아니라 상호 의존적인 과정이고, 사람과의 상호작용에서 그 결합 효과가 누적될 수 있다는 것이 논문의 출발점입니다.

---

## Related Work Map
CoGen은 단순한 RL 논문도 아니고, 단순한 pragmatic reasoning 논문도 아닙니다. 아래 여러 흐름의 교차점에 있습니다.

### A. Reference Games and Grounded Communication
- tangram reference game 계열
- speaker가 설명하고 listener가 target을 고르는 고전적 협업 과제

CoGen은 이 축을 실험 플랫폼으로 사용합니다. 중요한 점은 이 플랫폼이 단순 toy task가 아니라, generation과 comprehension의 상호작용을 정밀하게 분리해서 볼 수 있게 해준다는 점입니다.

### B. Pragmatic Reasoning / RSA
- Rational Speech Acts (RSA) 계열
- speaker와 listener가 서로를 모델링하는 재귀적 추론 프레임워크

CoGen의 `joint inference`는 이 흐름과 직접 연결됩니다. 다만 정적인 pragmatic model을 평가하는 데 그치지 않고, **사람과의 상호작용 속에서 시간이 지나며 어떤 변화가 누적되는가**를 봅니다.

### C. Continual Learning from Interaction
- interaction feedback으로 모델을 계속 업데이트하는 계열
- instruction following, question answering, interactive learning 연구와 인접

CoGen은 이 축을 vision-language reference game으로 옮기고, feedback 신호를 generation과 comprehension 양쪽에 동시에 연결합니다.

### D. RLHF and Preference Learning
- RLHF는 보통 외부 annotator의 pairwise preference를 사용

CoGen은 이와 다르게, **실제 상호작용의 성공/실패에서 얻는 binary feedback**을 사용합니다. 즉 preference label을 따로 수집하지 않고 interaction 자체를 학습 신호로 삼습니다.

정리하면 CoGen의 위치는 다음과 같습니다.
- 문제 측면: speaker/listener를 별도 task가 아닌 coupled process로 재정의
- 방법 측면: training-time data sharing + inference-time joint reasoning 결합
- 학습 측면: human interaction feedback 기반 continual learning

---

## 2) Task Setup: Reference Game and Feedback Signal
상호작용 단위는 2인 reference game입니다.

- context image set: \\(I = \{I_1, \ldots, I_N\}\\)
- target image: \\(I_t \in I\\)
- speaker utterance: \\(u\\)
- listener prediction: \\(\hat{t}\\)

게임 흐름:
1. speaker는 target \\(I_t\\)를 본다
2. speaker는 listener가 맞힐 수 있도록 utterance \\(u\\)를 생성한다
3. listener는 context 안에서 target을 고른다
4. 맞히면 성공, 틀리면 실패

핵심 포인트:
- 논문은 이 게임에서 **하나의 모델이 speaker와 listener 두 역할을 모두 수행**하게 한다
- 그리고 사람과 수천 번 상호작용하면서 점진적으로 모델을 업데이트한다

<figure class="media-panel">
  <img src="{{ '/assets/images/cogen-figure-1-reference-game.png' | relative_url }}" alt="Figure 1 from CoGen showing the reference game with speaker and listener roles" />
  <figcaption>
    <strong>Figure 1.</strong> 논문의 reference game 설정. speaker는 target image를 설명하고, listener는 그 설명을 바탕으로 context 안에서 target을 고른다.
  </figcaption>
</figure>

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
      <tr><td>\(I\)</td><td>Reference game context image set</td></tr>
      <tr><td>\(I_t\)</td><td>Target image in the context</td></tr>
      <tr><td>\(u\)</td><td>Speaker utterance / description</td></tr>
      <tr><td>\(\hat{t}\)</td><td>Listener prediction</td></tr>
      <tr><td>\(P_l(t \mid I, u; \theta)\)</td><td>Listener / comprehension distribution</td></tr>
      <tr><td>\(P_s(u \mid I, t; \theta)\)</td><td>Speaker / generation distribution</td></tr>
      <tr><td>\(D_{l,\rho}, D_{s,\rho}\)</td><td>Round \(\rho\) 에 수집된 comprehension / generation data</td></tr>
      <tr><td>\(r \in \{-1, 1\}\)</td><td>Interaction feedback reward</td></tr>
      <tr><td>\(\lambda_l, \lambda_s\)</td><td>Joint inference mixing coefficients</td></tr>
    </tbody>
  </table>
</div>

### Feedback가 정확히 무엇인가?
이 논문에서 feedback은 추상적인 preference가 아닙니다.

- listener가 맞히면 positive reward \\(r = 1\\)
- listener가 틀리면 negative reward \\(r = -1\\)

즉, **interaction outcome 자체가 학습 신호**입니다.

이 점이 중요합니다.
- RLHF처럼 외부 비교 라벨을 따로 모으지 않음
- task success/failure가 곧 feedback이 됨
- 그래서 “학습 신호가 자연스럽게 interaction에서 나온다”는 장점이 있음

---

## 3) Why Coupling Should Help
논문의 핵심 주장에는 두 갈래가 있습니다.

### 3.1 Training-Time Argument
generation만 feedback으로 학습하면, 모델은 자기 자신이 만든 utterance만 반복해서 보게 됩니다.

이 경우 생기는 위험:
- vocabulary collapse
- human-like expression에서 drift
- speaker는 잘 맞히게 만들더라도 표현이 점점 기계적이 될 수 있음

반면 comprehension 쪽 데이터는 사람의 utterance를 포함합니다. 따라서 comprehension 데이터를 generation 학습에도 공유하면, generation 모델이 인간 표현 분포를 계속 보게 됩니다.

### 3.2 Inference-Time Argument
listener가 잘 맞힐 utterance를 만들려면, speaker는 listener 관점에서 그 utterance가 얼마나 구별력 있는지도 고려해야 합니다.

즉 generation은 사실상 다음 질문을 품고 있어야 합니다.

> “내가 만든 설명을 listener가 이 context 안에서 실제로 target에 연결할 수 있는가?”

CoGen의 joint inference는 이 질문을 inference 단계에 직접 넣습니다.

---

## 4) CoGen Method (Technical Deep Dive)
논문의 방법은 크게 `training-time coupling`과 `inference-time coupling` 두 축으로 나뉩니다.

### 4.1 Continual Learning Loop
매 round \\(\rho\\)마다 시스템은 다음을 반복합니다.

1. 현재 모델을 사람과의 game에 배치
2. speaker / listener 역할에서 interaction data 수집
3. feedback으로 학습 데이터 구성
4. 누적 데이터로 모델 재학습
5. 업데이트된 모델을 다음 round에 다시 배치

즉, 모델은 고정된 offline dataset이 아니라 **자신이 만들어낸 상호작용과 그 결과**를 계속 먹으면서 진화합니다.

<figure class="media-panel">
  <img src="{{ '/assets/images/cogen-figure-2-coupled-learning.png' | relative_url }}" alt="Figure 2 from CoGen showing continual learning with coupled comprehension and generation" />
  <figcaption>
    <strong>Figure 2.</strong> continual learning 전체 루프. reference game 상호작용과 training이 번갈아 일어나고, 모델은 generation과 comprehension 두 역할을 모두 수행하면서 서로의 신호를 활용한다.
  </figcaption>
</figure>

### 4.2 Learning Objective
논문은 learning을 contextual bandit로 놓고, comprehension과 generation을 합친 additive objective를 사용합니다.

comprehension example \\((I, u, \hat{t}, r)\\)에 대한 gradient는 다음과 같습니다.

$$
\Delta_l = c_l \, r \, \nabla \log P_l(\hat{t} \mid I, u; \theta)
$$

여기서 \\(c_l\\)은 negative example이 loss를 과도하게 지배하지 않도록 넣은 cased inverse propensity score 계수입니다. generation 쪽도 같은 형태로, 분포만 \\(P_s(u \mid I, t; \theta)\\)로 바뀝니다.

세미나에서는 이렇게 이해하면 충분합니다.
- positive interaction은 그 행동을 강화
- negative interaction은 그 행동을 약화
- 다만 negative signal이 폭주하지 않게 importance correction을 넣음

### 4.3 Data Sharing: Training-Time Coupling
이 논문의 첫 번째 핵심 coupling은 `data sharing`입니다.

아이디어:
- listener 역할에서 얻은 positive example은 speaker 학습에도 유용
- speaker 역할에서 얻은 positive example은 listener 학습에도 유용

논문 식으로는:

$$
D_{l,\rho} = D_{l,\rho} \cup \{(I, \hat{u}, t, r) \in D_{s,\rho} \mid r = 1\}
$$

$$
D_{s,\rho} = D_{s,\rho} \cup \{(I, u, \hat{t}, r) \in D_{l,\rho} \mid r = 1\}
$$

해석:
- speaker가 낸 utterance로 게임이 성공했다면, 그 utterance는 해당 target에 대한 valid description이므로 listener 학습에도 쓸 수 있다
- listener가 사람의 utterance를 보고 정답을 맞혔다면, 그 utterance는 generation에도 도움이 되는 human expression example이다

왜 positive example만 공유하나?
- negative example은 실패 원인이 모호하다
- utterance가 나빴는지, listener가 실수했는지, ambiguity 때문인지 분리하기 어렵다
- 그래서 cross-task conversion에는 positive signal만 사용하는 편이 더 안정적이다

### 4.4 Joint Inference: Inference-Time Coupling
두 번째 coupling은 추론 단계에서 들어갑니다.

comprehension의 joint distribution은:

$$
P_l^{j}(t \mid I, u; \theta) =
\frac{
P_l(t \mid I, u; \theta)^{\lambda_l}
P_s(u \mid I, t; \theta)^{1-\lambda_l}
}{
\sum_{t'} P_l(t' \mid I, u; \theta)^{\lambda_l}
P_s(u \mid I, t'; \theta)^{1-\lambda_l}
}
$$

즉 listener는 “내 분포상 맞는 target인가?”만 보는 것이 아니라,

> “speaker 분포 관점에서도 이 utterance가 해당 target에서 나올 법한가?”

를 함께 본다는 뜻입니다.

generation 쪽도 같은 아이디어를 적용하지만, utterance space가 너무 커서 normalization이 어렵기 때문에 speaker 분포에서 샘플을 뽑아 re-ranking 근사로 처리합니다.

논문 구현:
- \\(\lambda_l = 0.5\\)
- \\(\lambda_s = 0\\)

표면적으로 \\(\lambda_s = 0\\)이면 generation에서 speaker term이 빠지는 것처럼 보이지만, 실제로는 **speaker 분포가 샘플 후보를 제공하는 source**로 남아 있어 영향을 계속 줍니다.

### 4.5 System Variants
논문 결과를 읽을 때는 variant 이름을 먼저 고정해야 합니다.

<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th>System</th>
        <th>Data Sharing</th>
        <th>Joint Inference</th>
        <th>의미</th>
      </tr>
    </thead>
    <tbody>
      <tr><td><strong>FULL</strong></td><td>Yes</td><td>Yes</td><td>제안 방법 전체</td></tr>
      <tr><td><strong>NO-DS</strong></td><td>No</td><td>Yes</td><td>training-time coupling 제거</td></tr>
      <tr><td><strong>NO-JI</strong></td><td>Yes</td><td>No</td><td>inference-time coupling 제거</td></tr>
      <tr><td><strong>BASELINE</strong></td><td>No</td><td>No</td><td>비결합 기준선</td></tr>
      <tr><td><strong>HUMAN</strong></td><td>-</td><td>-</td><td>사람-사람 interaction upper reference</td></tr>
    </tbody>
  </table>
</div>

### 4.6 Algorithm Sketch (Pseudocode)
<div class="pseudo-code">
  <p><strong>Input:</strong> initial VLM \(\theta_0\), human interaction rounds \(\rho = 1, \dots, R\)</p>
  <p>1) Deploy current model as both speaker and listener</p>
  <p>2) Collect interaction data \(D_{l,\rho}\) and \(D_{s,\rho}\) with binary rewards</p>
  <p>3) If using data sharing, convert positive examples across roles</p>
  <p>4) Re-train from initial weights on cumulative datasets \(\bigcup_{i \le \rho} D_{l,i}\) and \(\bigcup_{i \le \rho} D_{s,i}\)</p>
  <p>5) At inference, if enabled, use the opposing-role distribution for re-ranking</p>
  <p>6) Re-deploy updated model for the next round</p>
</div>

---

## 5) Experimental Setup
### 5.1 Environment and Data
- **Task**: two-player reference game
- **Visual domain**: KILOGRAM dataset의 abstract tangram shapes
- **Context size**: 10 images
- **Difficulty control**: CLIP similarity를 이용해 context 내 이미지가 서로 비슷하게 보이도록 구성

이 설정이 중요한 이유:
- 자연 이미지 캡셔닝보다 훨씬 더 fine-grained distinction이 필요
- 따라서 pragmatic generation과 precise comprehension을 보기에 적절함

### 5.2 Model and Initialization
- backbone: **IDEFICS2-8B**
- task는 prompting으로 구분
- first deployment 전에는 약 `10^4`개의 successful human-human games로 seed fine-tuning
- validation은 280 successful human-human games 사용

### 5.3 Deployment Protocol
- 총 **4 rounds**의 human deployment
- round 1: 역할별 2,000 interactions
- 이후 매 round마다 역할별 500 interactions 증가
- 총 MTurk 비용: **$12,980**

### 5.4 Metrics
- **Comprehension accuracy**: listener가 target을 맞힌 비율
- **Generation accuracy**: human listener가 speaker utterance를 듣고 target을 맞힌 비율
- **MAUVE**: 모델 언어가 human language distribution과 얼마나 비슷한지
- **SND (Shape Naming Divergence)**: 같은 tangram을 여러 context에서 얼마나 다양하게 설명하는지 측정하는 lexical diversity / pragmaticity proxy

여기서 SND는 처음 보면 생소할 수 있는데, 직관적으로는 다음과 같습니다.

> 같은 도형을 어떤 context에서는 “swan”, 다른 context에서는 “bird with long neck”처럼 다르게 표현할 수 있다면, context-sensitive pragmatic reasoning이 더 잘 작동하고 있다고 볼 수 있습니다.

---

## 6) Main Results
### 6.1 Performance Summary
논문의 가장 중요한 결과는 `coupling이 성능 향상 속도와 최종 성능 둘 다 개선한다`는 점입니다.

초기 round에서 joint inference 효과:
- comprehension: `53.31%` vs `42.64%`
- generation: `52.00%` vs `48.45%`

여기서 전자는 `FULL/NO-DS` 계열, 후자는 `NO-JI/BASELINE` 계열입니다. 즉 첫 라운드부터 **joint inference 유무가 바로 성능 차이로 드러납니다.**

최종 round에서 FULL:
- comprehension: `53.31 → 72.79%`
- generation: `52.00 → 78.07%`

absolute gain:
- comprehension: `+19.48`
- generation: `+26.07`

비결합 baseline 대비 최종 격차:
- comprehension: `+14.80`
- generation: `+17.10`

<figure class="media-panel">
  <img src="{{ '/assets/images/cogen-figure-3-performance.png' | relative_url }}" alt="Figure 3 from CoGen showing comprehension and generation accuracy across rounds" />
  <figcaption>
    <strong>Figure 3.</strong> 네 번의 deployment round 동안 FULL이 모든 ablation을 지속적으로 앞선다. caption의 핵심 메시지처럼, coupling은 단발성 gain이 아니라 학습 곡선 전체를 위로 올린다.
  </figcaption>
</figure>

### 6.2 Why 이 결과가 중요한가?
단순히 “더 잘 맞힌다”를 넘어서, sample efficiency가 좋아졌다는 점이 중요합니다.

논문은 FULL이 **baseline보다 1/3 이하의 human interactions만 가지고도 더 높은 성능**에 도달한다고 보고합니다. 즉 coupling은 단순 accuracy booster가 아니라, human feedback를 더 효율적으로 쓰게 만드는 장치입니다.

### 6.3 Ablation Reading
성능 곡선을 읽을 때는 두 coupling의 역할을 분리해서 봐야 합니다.

- **joint inference**
  - 즉각적인 성능 이득이 큼
  - 첫 deployment round부터 차이가 난다
  - “현재 모델이 이미 가진 speaker/listener 분포를 더 똑똑하게 사용한다”는 효과

- **data sharing**
  - language trend와 장기적 learning에 더 중요
  - 모델이 human utterance stream을 generation에 계속 주입받게 한다
  - vocabulary collapse와 human-language drift를 완화

즉 joint inference는 `fast effect`, data sharing은 `long-horizon effect`에 가깝다고 해석할 수 있습니다.

### 6.4 Language Analysis
이 논문이 좋은 점은 accuracy만 보지 않고, 모델 언어가 어떻게 바뀌는지도 함께 본다는 점입니다.

핵심 관찰:
- utterance length는 시간이 지나며 감소
  - 사람도 같은 경향을 보임
  - 협업이 진행되며 더 경제적인 표현을 쓰게 되는 reference game의 고전적 현상과 일치
- coupled variants는 effective vocabulary 감소가 덜 심함
- FULL은 매 round 가장 많은 새 단어를 추가
- FULL의 **MAUVE**는 시간이 지나며 증가
  - 즉 human language distribution과 더 가까워짐
- FULL은 **SND**도 가장 높음
  - context-sensitive pragmatic variation이 가장 큼

<figure class="media-panel">
  <img src="{{ '/assets/images/cogen-figure-5-language.png' | relative_url }}" alt="Figure 5 from CoGen showing language analysis trends" />
  <figcaption>
    <strong>Figure 5.</strong> utterance length, vocabulary, new words, MAUVE, SND를 같이 본 language analysis. FULL이 human-like language alignment와 pragmatic diversity를 가장 잘 유지한다는 점을 보여준다.
  </figcaption>
</figure>

이 결과가 말해주는 것:

> coupling은 성능만 올리는 것이 아니라, language를 더 human-like하고 pragmatic하게 만든다.

이 논문의 좋은 포인트는 여기입니다. 흔히 reinforcement-style adaptation은 accuracy는 올려도 language quality는 망가질 수 있는데, CoGen은 적어도 이 setting에서는 그 반대 방향의 증거를 보여줍니다.

---

## 7) Critical Appraisal
### Strengths
1. comprehension/generation coupling을 매우 명료한 실험 설계 안에 넣었다
2. 성능뿐 아니라 language trend까지 함께 측정했다
3. human-in-the-loop continual learning을 실제 규모로 수행했다

### What I Find Technically Interesting
가장 흥미로운 점은 **task coupling 자체가 regularizer처럼 작동한다**는 해석입니다.

generation이 자기 출력만 반복해서 먹으면 언어가 닫힌계로 수렴하기 쉬운데, comprehension data sharing이 그 폐쇄 루프를 열어줍니다. 즉 다른 task에서 들어오는 human language가 distributional anchor 역할을 합니다.

### Limitations
1. setting이 abstract tangram reference game이라 실제 open-domain dialogue로 바로 일반화되지는 않는다
2. worker population을 고정했기 때문에 새 사용자가 유입되는 실제 배포 환경과는 다르다
3. 학습은 round마다 cumulative data로 다시 학습하는 구조라 비용이 크다
4. generation 쪽 joint inference는 sampling approximation에 의존한다

### RLHF와의 차이
세미나에서 자주 나올 질문은 “이게 RLHF랑 뭐가 다른가?”입니다.

짧게 답하면:
- RLHF: 외부 annotator가 output pair를 비교해 preference 제공
- CoGen: interaction success/failure에서 binary reward를 직접 얻음

즉 CoGen은 preference learning이라기보다 **interactive bandit learning with coupled tasks**에 더 가깝습니다.

---

## 8) Seminar Takeaways
1. CoGen의 핵심은 generation과 comprehension을 따로 잘하게 만드는 것이 아니라, 둘이 서로를 학습 신호로 사용하게 만드는 데 있다.
2. coupling은 두 수준에서 일어난다: training-time `data sharing`, inference-time `joint inference`.
3. 결과는 accuracy 향상에 그치지 않고, language를 더 human-like하고 pragmatic하게 만든다.
4. 이 논문은 “feedback로부터 학습한다”는 말을, static preference collection이 아니라 실제 협업 상호작용의 성공/실패로 구현했다는 점에서 의미가 크다.

## Closing Line
CoGen은 “좋은 generator”와 “좋은 interpreter”를 따로 만드는 대신, 둘을 서로의 teacher로 삼는 구조가 장기적으로 더 강하다는 점을 보여줍니다. 그래서 이 논문은 multimodal agent를 설계할 때 **task decomposition보다 task coupling이 더 중요한 경우가 있다**는 메시지로 읽을 수 있습니다.
