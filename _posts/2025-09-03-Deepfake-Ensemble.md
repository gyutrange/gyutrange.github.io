---
layout: post
title: "윤리적 해커 양성 5기 수료 후기"
date: 2025-01-05
author: GiuNash
tags: [Education]
comments: true
toc: true
pinned: true
---
<style>
.caption {
    text-align: center;
    font-style: italic;
    color: gray;
}
</style>

# 주파수×공간×스펙트럼을 합친 소프트 보팅: 대회 데이터셋에서 AUC 1.000, Acc 96.7%

> - **아이디어**: 서로 다른 관점을 쓰는 3계열 모델(FreqNet·GenConViT·CaFft)에 소프트 보팅(가중 평균) 적용
> - **결과**: **Final Test** AUC **1.0000**, Accuracy **0.9667**, F1 **0.9789** / **FP=0**
> - **분석**: 가중치 학습 결과 **GenConViT**에 ~0.96이 몰림(주도권), 나머지는 미세 조정 역할
> - **교훈**: “강한 1개 + 보조 몇 개”가 **오탐 0 유지**에 유리, 단 **소수 FN**은 임계값/보정으로 더 줄일 여지

---

## 1) 배경과 문제 정의

대회에서 제공한 **딥페이크 식별** 데이터셋을 사용해, 서로 **서로 다른 단서**를 보는 모델을 가볍게 합치면 얼마나 멀리 갈 수 있는지 실험했습니다.

- **FreqNet**: 위·변조 과정에서 생기는 **주파수 도메인** 아티팩트에 강점
- **GenConViT**: **공간적 패턴 + 전역 문맥**을 잘 잡는 비전 트랜스포머 계열
- **CaFft**: **FFT 기반 스펙트럼 주의**로 주기/텍스처 왜곡을 포착

핵심은 **엔SEMBLE이 복잡할 필요 없다**는 점입니다. 각 모델의 **fake 확률**을 **가중 평균**(soft voting)만 해도 꽤 강력합니다.

---

## 2) 방법: 소프트 보팅(가중 평균)

### 2.1 수식

확률 P^i 를 내는 N개의 베이스 모델이 있을 때, 최종 예측 확률 P^ 는

<img width="433" height="91" alt="Screenshot 2025-09-03 at 3 54 28 PM" src="https://github.com/user-attachments/assets/a3ab034c-d503-4a84-a45b-8322122862da" />


로 계산했습니다.

**가중치 www** 는 검증 세트에서 **로스(로그로스)** 최소화 기준으로 학습했고, **음수 금지 + 합 1** 제약을 뒀습니다(간단·안정).

### 2.2 가중치 학습 결과

- (5모델 실험에서) 학습된 가중치 예시:
    - **model_B (GenConViT)**: **0.9606** · **0.9613**(다른 러닝)
    - 나머지(FreqNet/CaFft/face-x-ray/cnn-rnn): **0.005~0.016** 수준의 미세 가중
- 해석: **GenConViT가 주도**, FreqNet/CaFft 등은 **엣지 케이스 보정/캘리브레이션** 역할

> 참고: 본문의 핵심 아이디어는 3모델(FreqNet·GenConViT·CaFft) 이지만, 검증 단계에서 legacy 탐지기(face-x-ray, cnn-rnn) 를 함께 투입해도 학습된 가중치가 거의 0에 가까워 과적합을 크게 유발하지 않음을 확인했습니다.
> 

---

## 3) 데이터 & 평가 설정

- **데이터**: 대회 제공 데이터셋 (train/valid는 내부 분할, **최종 평가는 주최측 test set**)
- **평가 지표**: AUC, Accuracy, F1, Precision/Recall, Confusion Matrix
- **임계값**: 기본 **0.5**

---

## 4) 결과

### 4.1 Final Test Set

- **AUC**: **1.0000**
- **Accuracy**: **0.9667**
- **F1**: **0.9789**

**Confusion Matrix (threshold=0.5)**

- **TP (Fake→Fake)**: 93
- **TN (Real→Real)**: 23
- **FP (Real→Fake)**: **0**
- **FN (Fake→Real)**: 4
- **Fake acc**: 0.9588 / **Real acc**: **1.0000**

> 오탐(Real을 Fake로) 0건이 인상적입니다. 운영 관점에서 인간/정책 리뷰 비용을 크게 줄여줍니다.
> 

---

### 4.2 내부 검증(399 샘플)

- **AUC**: **0.9994**
- **Accuracy**: **0.9424**
- **F1**: **0.9630**
- **Precision**: **1.0000**
- **Recall**: **0.9286**

**Confusion Matrix**

- **TN**: 77 / **FP**: **0**
- **FN**: 23 / **TP**: 299

> 일관되게 FP=0, FN>0 패턴. 즉, “보수적” 모델: 진짜(Real)를 가짜로 몰지 않지만, 몇몇 Fake를 놓침. 이는 임계값/보정으로 개선 여지가 있습니다.
> 

---

## 5) 에러 분석: 놓친 Fake들은 왜?

샘플 일부를 보면,

- `abofeumbvv.mp4`: **FreqNet=0.9903**(강한 fake), **GenConViT=0.50**, **CaFft=0.4897**, **ensemble=0.4828 → 0(Real)**
    
    → **주도 가중치가 GenConViT** 에 쏠리다 보니, FreqNet의 강한 시그널이 **덮였**습니다.
    

이런 류의 **FN(가짜를 실수로 진짜로)** 을 줄이려면:

1. **임계값을 0.5→0.45 등으로 조금 낮추기**: FP=0 특성이 유지되는 범위에서 **Recall↑**
2. **가중치 스무딩**: 주도 모델의 가중 상한을 두거나, 보조 모델에 **최소 바닥 가중치** 부여
3. **스태킹(Logistic Regression/GBDT 메타모델)**: 입력에 **신뢰도/품질 지표**(얼굴 검출 스코어, 프레임 수 등)를 추가하면, **경계 샘플**에서 더 유연한 결정을 합니다.

---

## 6) 구현 메모 (간단 코드 스니펫)

### 6.1 소프트 보팅

```python
import numpy as np

# p_list: [FreqNet, GenConViT, CaFft, face_xray, cnn_rnn] 확률 배열 (N, 5)
# w: 학습된 가중치 (합=1, 음수금지)
def soft_vote(p_list, w):
    p = np.clip(p_list, 0.0, 1.0)     # 안전장치: -1.0 등 이상치 클램프
    w = np.array(w, dtype=np.float64)
    w = np.maximum(w, 0); w /= (w.sum() + 1e-12)
    return (p * w).sum(axis=1)

```

### 6.2 가중치 학습(개념)

```python
# 검증셋 (P: (N, K) 모델 확률, y: (N,) 레이블)
# 목적: nonneg & sum=1 제약하 Cross-Entropy 최소화
# 실전에서는 L-BFGS-B + softmax 파라미터화(=무제약)로 간단히 해결 가능

```

---

## 7) 작은 팁: 운영(Production) 관점

- **Calibration(확률 보정)**: Platt/Temperature Scaling으로 **경계 샘플 안정화**
- **Threshold per slice**: **도메인/조명/해상도** 별로 임계값을 미세 조정
- **Fail-safe 루틴**: `face-x-ray`처럼 **이상 확률(-1.0)** 이 튀는 입력은 **클램프·재시도·로깅**
- **라벨 노이즈 완화**: 주파수·스펙트럼 편향 샘플이 많은 경우, **데이터 균형·증강**이 효과적

---

## 8) 원시 결과

<img width="916" height="713" alt="Screenshot 2025-09-03 at 3 58 19 PM" src="https://github.com/user-attachments/assets/3217b78f-abfb-4668-862d-5cf46955e952" />


```
('aagfhgtpmv.mp4', 0.9286), ('aapnvogymq.mp4', 0.7439),
('abarnvbtwb.mp4', 0.0772), ('abofeumbvv.mp4', 0.4828),
('abqwwspghj.mp4', 0.9592), ...

```

- 경계치(≈0.48~0.52) 근처 샘플이 **FN/TP 뒤바뀜의 주범**입니다. 위 6장 항목(임계값/보정/스태킹)으로 다뤄볼 영역.

---

## 9) 결론과 다음 단계

- **한 줄 요약**: **강한 1개(GenConViT) + 보조(FreqNet/CaFft)** 만으로도 **AUC 1.0 / Acc 96.7% / FP=0** 를 달성.
- **다음 단계**
    1. **임계값 스윕 & 캘리브레이션**으로 **FN 추가 감축**
    2. **스태킹 메타모델** 도입(품질 피처 포함)
    3. **하드/소프트 하이브리드**: 특정 조건(저조도/블러)에서 **보팅 규칙을 전환**
    4. **데이터 슬라이스별 분석**으로 **도메인 편향** 교정

> 실험의 메시지는 분명합니다. “복잡한 앙상블”이 아니라도, 관점이 다른 모델 몇 개를 ‘제대로’ 섞으면 운영 친화적인 성능(특히 오탐 0)을 만들 수 있다. 남은 과제는 놓친 가짜(FN) 를 운영 리스크 허용 범위 내에서 더 줄이는 일입니다.
> 

---

### 부록 A. 표로 보는 핵심 수치

| Split | AUC | Acc | F1 | Precision | Recall | FP | FN |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Final Test** | **1.0000** | **0.9667** | **0.9789** | – | – | **0** | 4 |
| **Validation(399)** | 0.9994 | 0.9424 | 0.9630 | **1.0000** | 0.9286 | **0** | 23 |
