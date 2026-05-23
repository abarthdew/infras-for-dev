# AI Basics

## 목차
- [AI의 범위](#ai의-범위)
- [Model](#model)
- [Training과 Inference](#training과-inference)
- [Dataset](#dataset)
- [Evaluation](#evaluation)
- [Overfitting](#overfitting)

---

## AI의 범위

### "AI, ML, DL은 범위가 어떻게 다른가?"

**AI는 가장 넓은 개념이고, ML은 AI의 부분집합, DL은 ML의 부분집합**이다.

```text
AI (Artificial Intelligence) 가장 넓음
├─ 규칙 기반 시스템
├─ 전문가 시스템
└─ Machine Learning
   ├─ 선형 회귀, SVM
   ├─ 의사결정 트리
   └─ Deep Learning (신경망) 가장 좁음
      ├─ CNN (이미지)
      ├─ RNN (시계열)
      └─ Transformer (언어)
```

### 각 분야의 특징

```text
AI (지능적 행동):
- 목표: 지능적 작업 수행
- 방법: 다양 (규칙, 통계, 학습)
- 예: 체스 엔진, 번역기

ML (데이터 학습):
- 목표: 데이터 패턴 학습
- 방법: 통계, 최적화
- 예: 스팸 필터, 추천 시스템

DL (신경망):
- 목표: 복잡한 패턴 학습
- 방법: 다층 신경망
- 예: 이미지 분류, 음성 인식
```

---

## Model

### Model이란?

```text
정의: 데이터 패턴을 수학적으로 표현한 함수

y = f(x)
where f is the trained model

예시:
- 선형 회귀: y = w1*x1 + w2*x2 + b
- 신경망: 여러 계층의 함수 합성
```

### Model 종류

```text
Supervised Learning (지도학습):
- 입력 X, 정답 Y 있음
- 예: 스팸 분류 (이메일 → 스팸/정상)
- 회귀: 연속값 예측 (집값 예측)
- 분류: 카테고리 예측 (고양이/개 분류)

Unsupervised Learning (비지도학습):
- 입력 X만 있음, 정답 없음
- 예: 고객 세분화 (비슷한 사람 묶음)
- 클러스터링, 차원 축소

Reinforcement Learning (강화학습):
- 보상 신호로 학습
- 예: 게임 플레이, 로봇 제어
```

---

## Training과 Inference

### "학습과 추론은 무엇이 다른가?"

**Training은 데이터에서 패턴을 학습해 모델을 만드는 과정, Inference는 만든 모델을 실제로 사용하는 과정**이다.

```text
Training (학습):
- 목표: 최적의 가중치 찾기
- 입력: 훈련 데이터 (수백만 샘플)
- 출력: 학습된 모델 (가중치)
- 시간: 느림 (몇 시간~몇 주)
- 빈도: 가끔 (모델 업데이트 시)
- 리소스: 높음 (GPU 필요)

Inference (추론):
- 목표: 새로운 입력 예측
- 입력: 새로운 샘플 (1개 또는 배치)
- 출력: 예측값
- 시간: 빠름 (밀리초)
- 빈도: 자주 (매초 수만 번)
- 리소스: 낮음 (CPU로도 가능)
```

### Training 과정

```text
1. 데이터 준비
   - 특징 추출 (Feature Engineering)
   - 정규화 (Normalization)

2. 모델 초기화
   - 가중치 무작위 설정

3. 반복 (Epoch)
   for epoch in 1 to max_epochs:
       for batch in training_data:
           predictions = model(batch)
           loss = compute_loss(predictions, labels)
           gradients = compute_gradients(loss)
           update_weights(gradients)
           
4. 모델 저장
   - 최종 가중치 저장
```

### Inference 과정

```text
loaded_model = load_model('model.pkl')

new_email = "Click here to win $1000!"
prediction = loaded_model.predict(new_email)
# output: "spam" (확률 0.95)
```

---

## Dataset

### "train/test split은 왜 필요한가?"

**Train/Test Split은 모델이 새로운 데이터에 잘 작동하는지 확인하기 위해 필요**하다. 전체 데이터로 학습하면 overfitting을 감지할 수 없다.

```text
전체 데이터 1000개
├─ Training Set (80% = 800개) → 모델 학습
└─ Test Set (20% = 200개) → 성능 평가

Train Accuracy: 99% (학습했으니 당연)
Test Accuracy: 85% (새로운 데이터)
→ Overfitting 의심!
```

### Data Split 전략

```text
기본: Train 70%, Validation 15%, Test 15%

Train Set:
- 모델 학습에 사용
- 가중치 업데이트

Validation Set:
- 학습 중 성능 평가
- Hyperparameter 튜닝
- Early stopping

Test Set:
- 최종 평가만 (학습 중 절대 사용 X)
- 모델 성능의 객관적 지표
```

### 데이터 불균형

```text
문제: 클래스 불균형
- 정상 이메일 99%
- 스팸 이메일 1%

모델이 "모두 정상"이라고 예측 → 정확도 99%
하지만 스팸 탐지 못함!

해결책:
✓ Oversampling (소수 클래스 복제)
✓ Undersampling (다수 클래스 감소)
✓ SMOTE (합성 샘플 생성)
✓ Class weight (손실함수에 가중치)
```

---

## Evaluation

### 성능 평가 지표

```text
분류 (Classification):
- Accuracy: 정확히 맞힌 비율
- Precision: 양성이라 한 것 중 정답 비율
- Recall: 실제 양성 중 맞힌 비율
- F1-Score: Precision과 Recall의 조화평균

회귀 (Regression):
- MSE (Mean Squared Error)
- RMSE (Root Mean Squared Error)
- MAE (Mean Absolute Error)
- R² (설명력)
```

### Confusion Matrix

```text
분류: Spam vs Not Spam

         Predicted Spam  Predicted Not
Actual Spam      TP(80)          FN(20)
Actual Not       FP(10)          TN(890)

Precision = TP / (TP + FP) = 80/90 = 0.89
Recall = TP / (TP + FN) = 80/100 = 0.80
Accuracy = (TP + TN) / Total = 970/1000 = 0.97
```

---

## Overfitting

### "overfitting은 왜 문제가 되는가?"

**Overfitting은 모델이 훈련 데이터에만 과도하게 최적화되어 새로운 데이터에 잘못 작동**한다.

```text
이상적 모델:
Train Accuracy: 85%
Test Accuracy: 84%
→ 새로운 데이터에도 잘 작동

Overfitting:
Train Accuracy: 99%
Test Accuracy: 60%
→ 훈련 데이터의 잡음까지 학습

Underfitting:
Train Accuracy: 70%
Test Accuracy: 70%
→ 패턴을 제대로 못 배움
```

### Overfitting 방지

```text
1. 더 많은 데이터
   - 잡음의 영향 감소

2. 정규화 (Regularization)
   - L1, L2 정규화
   - 가중치 크기 제한

3. Dropout (신경망)
   - 훈련 중 일부 뉴런 비활성화
   - 다양한 부분 네트워크 학습

4. Early Stopping
   - Validation loss가 증가하면 학습 중단
   - 과도한 훈련 방지

5. Cross Validation
   - 데이터를 여러 부분으로 나눠 평가
   - 더 신뢰성 높은 성능 추정
```

### 시각화

```text
Bias-Variance Trade-off:

High Bias, Low Variance (Underfitting):
- 모델이 너무 단순함
- 훈련/테스트 모두 성능 나쁨

Low Bias, High Variance (Overfitting):
- 모델이 너무 복잡함
- 훈련 성능 좋음, 테스트 성능 나쁨

Low Bias, Low Variance (이상):
- 모델 복잡도 적절
- 훈련/테스트 성능 모두 좋음
```

---

## 정리

| 개념 | 설명 |
|------|------|
| AI | 지능적 행동을 수행하는 시스템 |
| ML | 데이터에서 패턴 학습 |
| DL | 신경망을 이용한 패턴 학습 |
| Training | 데이터에서 모델 학습 |
| Inference | 학습된 모델로 예측 |
| Overfitting | 훈련 데이터에 과도하게 최적화 |

---

## Related Notes

- [Machine Learning](../machine-learning/overview.md)
- [LLM](../llm/overview.md)
