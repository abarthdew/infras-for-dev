# Machine Learning

## 목차
- [Machine Learning의 역할](#machine-learning의-역할)
- [Supervised Learning](#supervised-learning)
- [Unsupervised Learning](#unsupervised-learning)
- [Feature](#feature)
- [Loss Function](#loss-function)
- [Optimization](#optimization)
- [Evaluation Metric](#evaluation-metric)

---

## Machine Learning의 역할

**Machine Learning은 데이터에서 패턴을 학습해 새로운 데이터에 대해 예측이나 분류를 수행하는 방법**이다. 명시적인 규칙을 프로그래밍하지 않고 데이터 자체에서 규칙을 추출한다.

```text
Traditional Programming:
규칙 작성 → 데이터 입력 → 결과 출력
("if-else" 로직)

Machine Learning:
데이터 입력 → 패턴 학습 → 규칙 자동 추출 → 결과 출력
(자동으로 규칙 생성)
```

### 기본 흐름

```text
Training Phase:
특성 추출 (Features)
    ↓
모델 학습 (손실 최소화)
    ↓
가중치 최적화 (Optimization)
    ↓
학습된 모델

Inference Phase:
새 데이터 입력
    ↓
학습된 모델 적용
    ↓
예측 결과 출력
```

---

## Supervised Learning

### "supervised learning과 unsupervised learning은 무엇이 다른가?"

**Supervised Learning은 입력 X와 정답 Y가 있어서 X→Y 매핑을 학습하는 방식, Unsupervised Learning은 X만 있어서 패턴을 스스로 찾는 방식**이다.

```text
Supervised Learning (지도학습):
입력(Features) + 정답(Label) → 모델 학습

예:
- 이메일 → 스팸/정상 (분류)
- 집의 평방피트 → 가격 (회귀)
- 영상 픽셀 → 고양이/개 (분류)
```

### Classification vs Regression

```text
Classification (분류):
- 정답: 카테고리 (종속)
- 출력: 클래스 또는 확률
- 예: 스팸(0) vs 정상(1)
- 지표: Accuracy, Precision, Recall, F1

회귀 (Regression):
- 정답: 연속값
- 출력: 실수
- 예: 집값 예측 (100만원, 200만원, ...)
- 지표: MSE, RMSE, MAE, R²
```

### 구현 예시

```python
# Sklearn 분류 (Logistic Regression)
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# 데이터 준비
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 모델 학습
model = LogisticRegression()
model.fit(X_train, y_train)  # 패턴 학습

# 예측
y_pred = model.predict(X_test)

# 평가
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.2%}")
```

---

## Unsupervised Learning

### "unsupervised learning은 어떤 경우에 쓰는가?"

**Unsupervised Learning은 정답 없이 데이터의 숨은 구조나 패턴을 찾아내는 방식**이다. 레이블 없는 데이터가 많을 때 유용하다.

```text
Unsupervised Learning (비지도학습):
입력(Features만) → 패턴 자동 발견

예:
- 고객 세분화 (비슷한 구매 패턴인 고객 묶음)
- 이상 탐지 (정상과 다른 거래 탐지)
- 차원 축소 (고차원 데이터 단순화)
- 추천 시스템 (유사한 상품 찾기)
```

### 주요 기법

```text
Clustering (클러스터링):
- 비슷한 데이터 묶음
- K-Means: 미리 정한 개수 K로 분할
- DBSCAN: 밀도 기반 묶음
- Hierarchical: 계층 구조

예:
고객 1: [구매액 높음, 방문 자주]
고객 2: [구매액 높음, 방문 자주]
고객 3: [구매액 낮음, 방문 드문]
→ Clustering → {고객 1, 2}, {고객 3}

Dimensionality Reduction (차원 축소):
- 고차원 → 저차원
- 변수 1000개 → 10개
- 계산 비용 절감, 시각화

이상 탐지 (Anomaly Detection):
- 정상과 다른 패턴
- 신용카드 이상 거래
- 네트워크 침입 탐지
```

### K-Means 예시

```python
from sklearn.cluster import KMeans

# 클러스터링
kmeans = KMeans(n_clusters=3)  # 3개 그룹으로 분할
kmeans.fit(X)  # 데이터 학습

# 각 샘플의 클러스터 (0, 1, 2)
labels = kmeans.labels_
print(labels)  # [0, 1, 2, 0, 1, ...]
```

---

## Feature

### "feature engineering은 왜 중요한가?"

**Feature Engineering은 원본 데이터에서 모델 학습에 유용한 특성들을 뽑아내는 과정**이다. 좋은 특성이 없으면 아무리 좋은 모델도 성능이 떨어진다.

```text
원본 데이터:
- "2026-05-23" (날짜)
- "서울 강남구" (주소)
- "$150,000" (금액 텍스트)

특성 공학:
- 날짜 → [월, 요일, 계절]
- 주소 → [위도, 경도, 강남=1/아니면=0]
- 금액 → 150000 (숫자 변환)
→ 모델이 사용 가능한 형태로 변환
```

### 특성 공학 기법

```text
1. 수치 특성 (Numerical Features)
   정규화 (Normalization): 0~1 범위로 스케일
   표준화 (Standardization): 평균 0, 표준편차 1

2. 범주 특성 (Categorical Features)
   원-핫 인코딩: [남성=1,0] [여성=0,1]
   라벨 인코딩: [남성=0] [여성=1]

3. 시간 특성 (Temporal Features)
   날짜 → [연도, 월, 일, 요일]
   시계열 → 이전 값 (lag), 이동 평균

4. 특성 상호작용 (Interaction)
   가격 × 개수 = 총액
   온도 × 습도 = 열지수
```

### 구현 예시

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler

# 데이터 준비
df = pd.DataFrame({
    'age': [25, 30, 35],
    'salary': [30000, 50000, 100000],
    'city': ['Seoul', 'Busan', 'Seoul']
})

# 수치 특성 정규화
scaler = StandardScaler()
df['age_scaled'] = scaler.fit_transform(df[['age']])

# 범주 특성 원-핫 인코딩
df = pd.get_dummies(df, columns=['city'])
print(df)
```

---

## Loss Function

### "loss와 metric은 왜 다를 수 있는가?"

**Loss는 모델 학습 중 최소화하는 목표 함수, Metric은 최종 성능 평가 지표**이다. 목표와 평가 관점이 다를 수 있어서 다르게 선택할 수 있다.

```text
Loss Function (학습 중):
- 목표: 가중치 업데이트를 위해 최소화할 값
- 미분 가능해야 함 (gradient descent)
- 예: MSE, Cross Entropy

Metric (평가 시):
- 목표: 모델 성능 평가
- 반드시 미분 가능할 필요 없음
- 사람이 이해하기 쉬워야 함
```

### Loss 함수 종류

```text
회귀 (Regression):
MSE (Mean Squared Error):
loss = 1/n * Σ(y_true - y_pred)²
→ 큰 오차에 페널티 (이상치 민감)

MAE (Mean Absolute Error):
loss = 1/n * Σ|y_true - y_pred|
→ 절댓값 (이상치 둔감)

분류 (Classification):
Cross Entropy:
loss = -Σ(y_true * log(y_pred))
→ 확률 분포 간 거리
→ 멀수록 손실 커짐

Hinge Loss:
loss = max(0, 1 - y_true * y_pred)
→ SVM에서 사용
```

### 예시: Loss vs Metric

```text
작업: 암 진단 분류

Loss = Cross Entropy
(학습 중 최소화)

Metric (평가):
- Accuracy: 전체 맞춘 비율
  BUT: 실제 암 5%, 정상 95%
  "모두 정상" 예측 → 95% 정확도 (틀린 평가!)
  
- Recall: 암 중 찾은 비율 (놓친 암 없나?)
  실제 암 100명 중 95명 진단
  → Recall = 95% (더 나은 지표)
```

---

## Optimization

### "모델은 어떻게 최적화되는가?"

**Optimization은 손실 함수를 최소화하는 가중치를 찾는 과정**이다. 기울기(gradient)를 이용해 가중치를 반복적으로 업데이트한다.

```text
목표:
최소 손실 찾기

방법: Gradient Descent (경사 하강법)
1. 현재 가중치에서 손실 계산
2. 손실의 기울기(gradient) 계산
3. 기울기 반대 방향으로 가중치 업데이트
4. 반복 (수렴할 때까지)

비유:
산을 내려가는데 안개 속에서 발을 디딤
현재 위치의 기울기를 느끼고
기울기 반대로 한 발 내디딤
계속 반복하면 계곡(최소값)에 도달
```

### 최적화 알고리즘

```text
SGD (Stochastic Gradient Descent):
- 한 샘플씩 업데이트
- 빠르지만 진동 (noisy)
- 시간: 빠름, 안정성: 낮음

Mini-batch GD:
- 작은 배치 (32, 64개)씩 업데이트
- 균형 잡힘
- 시간: 중간, 안정성: 중간

Adam (Adaptive Moment Estimation):
- 최신 최적화 알고리즘
- 기울기 방향과 크기 모두 고려
- 시간: 중간, 안정성: 높음
- 대부분 기본값으로 사용
```

### 구현 예시

```python
import torch
from torch.optim import Adam

# 모델과 옵티마이저 정의
model = MyModel()
optimizer = Adam(model.parameters(), lr=0.001)

# 학습 루프
for epoch in range(10):
    for X_batch, y_batch in dataloader:
        # 1. 예측
        y_pred = model(X_batch)
        
        # 2. 손실 계산
        loss = loss_fn(y_pred, y_batch)
        
        # 3. 기울기 계산
        optimizer.zero_grad()  # 이전 기울기 삭제
        loss.backward()  # gradient 계산
        
        # 4. 가중치 업데이트
        optimizer.step()  # 가중치 업데이트
```

---

## Evaluation Metric

### "평가 지표는 어떻게 선택해야 하는가?"

**평가 지표는 문제 특성에 따라 다르게 선택**해야 한다. 정확도(Accuracy)만으로는 클래스 불균형 문제를 제대로 평가할 수 없다.

```text
Precision과 Recall의 Trade-off:

Precision (정밀도):
"스팸이라고 한 것 중 정말 스팸인 비율"
= TP / (TP + FP)

Recall (재현율):
"실제 스팸 중 찾아낸 비율"  
= TP / (TP + FN)

선택:
- 스팸 필터: Recall 중요 (스팸을 못 거르면 안 됨)
- 승인 시스템: Precision 중요 (거짓 승인 위험)
- 암 진단: Recall 중요 (암을 못 찾으면 생명 위험)
```

### 평가 지표 선택 가이드

```text
분류 (Classification):

클래스 균형 O:
→ Accuracy

클래스 불균형 X (특히 소수 클래스 중요):
→ Precision, Recall, F1, AUC-ROC

다중 클래스:
→ Macro F1 (모든 클래스 동등)
→ Weighted F1 (빈도 고려)

회귀 (Regression):

일반:
→ RMSE, MAE

이상치 중요:
→ MAE (덜 민감)

정규화 필요:
→ MAPE (백분율 오차)
```

### 혼동 행렬과 지표

```text
         예측 양성  예측 음성
실제 양성  TP(80)   FN(20)
실제 음성  FP(10)   TN(890)

Accuracy = (TP+TN)/(Total) = 970/1000 = 97%

Precision = TP/(TP+FP) = 80/90 = 89%
(양성이라 예측한 것 중 맞은 비율)

Recall = TP/(TP+FN) = 80/100 = 80%
(실제 양성 중 찾은 비율)

F1 = 2*(Precision*Recall)/(Precision+Recall) = 84%
(Precision과 Recall의 조화평균)
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Supervised | 정답이 있는 학습 (분류, 회귀) |
| Unsupervised | 정답 없이 패턴 발견 (클러스터링, 차원축소) |
| Feature | 모델 입력으로 사용할 특성 |
| Feature Engineering | 원본 데이터에서 유용한 특성 추출 |
| Loss | 학습 중 최소화할 목표 함수 |
| Metric | 최종 성능 평가 지표 |
| Optimization | 손실 최소화 위한 가중치 조정 |
| Gradient Descent | 기울기를 이용한 최적화 알고리즘 |

---

## Related Notes

- [AI Basics](../ai-basics/overview.md)
- [LLM](../llm/overview.md)
- [Prompt Engineering](../prompt-engineering/overview.md)
