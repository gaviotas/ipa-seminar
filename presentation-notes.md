---
layout: page
title: "Presentation Notes"
permalink: /presentation-notes/
---

## 발표 목적
- 논문 원문을 따로 열지 않아도, 본 세미나 자료만으로 문제 정의부터 방법론/실험/한계까지 전달

## 15분 세미나 구성 (슬라이드 12장)

권장 시간 배분:
- Intro + Problem: 3분
- Method: 5분
- Experiments + Ablation: 5분
- Limitations + Q&A setup: 2분

### Slide 1. Title & One-liner
- 핵심 문장: "이 논문은 memorability를 예측하는 연구가 아니라 개선 행동을 생성하는 연구다."
- 멘트: 오늘은 MemFeed 문제정의, MemCoach 알고리즘, MemBench 평가설계를 순서대로 보겠습니다.

### Slide 2. Why This Problem
- 기존 패러다임: prediction / editing
- gap: capture-time 사용자 의사결정 지원 부재
- 멘트: 현장에서 필요한 것은 점수보다 다음 행동입니다.

### Slide 3. Task Definition (MemFeed)
- \(m_S = M(x_S),\ m_D = M(x_D),\ m_D > m_S\)
- actionable/interpretable/memorability-oriented 요구
- 주의: \(x_D\)는 추론 시 입력이 아니라 benchmark/training construction에서만 쓰이는 target view
- 멘트: 정답 텍스트 생성이 아니라 행동 유도 문제로 봐야 합니다.

### Slide 4. Why Not Plain MLLM
- LaMem: 사람 기반 memorability score가 달린 대표 benchmark
- 측정 방식: yes/no memorability 질문 후 token likelihood로 ranking을 만들고, GT ranking과 Spearman 비교
- inter-annotator 0.68 = 사람끼리의 memorability judgment 일치도 ceiling
- zero-shot IR 개선 한계
- IR = 편집 후 memorability가 원본보다 커진 비율
- editing baseline = empty feedback를 넣은 편집 결과
- 멘트: VLM general capability만으로 memorability direction은 약합니다.

### Slide 5. MemCoach Overview
- Contrasting generation → Steering extraction → Inference steering
- 멘트: teacher privileged signal을 student latent direction으로 distill합니다.

### Slide 6. Contrasting Pair Construction
- \(f_i^+ = \phi_{\text{teach}}(x_S^i, x_D^i, p_a)\)
- \(f_i^- = \phi_{\text{stud}}(x_S^i, p_m)\)
- 멘트: 같은 source에서 aware vs neutral feedback을 대조합니다.

### Slide 7. Steering Equations
- \(r^{(l)} = \frac{1}{N}\sum_i\left(h_{+,i}^{(l)} - h_{-,i}^{(l)}\right)\)
- \(\tilde{h}^{(l)} = h^{(l)} + \alpha r^{(l)}\)
- 기본값: \(l=12,\ \alpha=55\)
- 멘트: 파라미터 업데이트 대신 activation 이동으로 행동을 바꿉니다.

### Slide 8. MemBench and Protocol
- scene-based ranking pair construction
- metric: IR / RM / PPL
- 멘트: 문장 품질과 실제 개선 효과를 분리해서 봅니다.

### Slide 9. Main Results (Table 2)
- MemCoach(InternVL): IR 0.80 / RM 7.21 / PPL 4.99
- Zero-shot InternVL 대비 동시 개선
- 멘트: training-free인데도 zero-shot 대비 일관된 이득이 납니다.

### Slide 10. Generalization (Table 3)
- LLAVA, IDEFICS, QWEN, INTERNVL 모두 IR 개선
- 멘트: backbone-specific trick이 아니라 method-level gain입니다.

### Slide 11. Ablation & Failure Modes
- low-data 효율, α saturation, diff-operator 중요성
- Figure 7 failure case: out-of-context object를 제거하면 distinctiveness까지 사라져 RM이 오히려 떨어질 수 있음
- 멘트: 성능 향상과 함께 어떤 시각적 다양성을 희생하는지도 봐야 합니다. 이 논문은 '깔끔함'과 '기억남음'이 항상 같은 방향이 아니라는 점을 보여줍니다.

### Slide 12. Takeaways & Open Questions
- objective 전환의 의미
- personalization/human study 필요
- 멘트: 이 논문은 memorability 연구를 interactive guidance로 확장합니다.

## 예상 Q&A (박사 청중)
1. **Q. 왜 RLHF나 finetuning 대신 steering인가?**  
A. 비용/적용성 측면에서 training-free가 빠르고, 여러 backbone에 이식성이 높습니다.

2. **Q. 평가지표가 predictor/editor에 종속적이지 않나?**  
A. 맞습니다. 그래서 predictor/editor 교체 실험과 human study 보강이 후속 필수 과제입니다.

3. **Q. aesthetics feedback과 본질 차이는?**  
A. aesthetics는 시각적 선호 중심, MemFeed는 future recall 향상이라는 목적 함수가 다릅니다.

4. **Q. teacher oracle 격차는 어떻게 해석해야 하나?**  
A. oracle은 target view 정보를 직접 보므로 상한선에 가깝고, MemCoach는 그 정보를 latent direction으로 근사합니다.

## 발표 직전 체크리스트
1. 방법 섹션에서 식 2개를 정확히 판서/설명할 수 있는지
2. IR/RM/PPL 정의를 혼동 없이 말할 수 있는지
3. "왜 이 문제가 prediction보다 중요한가"를 30초 내 설명 가능한지

## 백업 슬라이드 제안
1. Eq.(3), Eq.(4)를 한 장에 묶은 수식 슬라이드
2. Table 2를 baseline family별로 색상 강조한 슬라이드
3. Figure 7 failure case만 따로 떼어낸 토론 슬라이드
