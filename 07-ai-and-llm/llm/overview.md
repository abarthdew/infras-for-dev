# LLM

## 목차
- [LLM의 역할](#llm의-역할)
- [Token](#token)
- [Context Window](#context-window)
- [Pretraining](#pretraining)
- [Instruction Tuning](#instruction-tuning)
- [Inference](#inference)
- [Hallucination](#hallucination)

---

## LLM의 역할

**LLM (Large Language Model)은 수십억 개의 파라미터를 가진 신경망으로, 대량의 텍스트에서 언어의 패턴을 학습해 텍스트를 생성**한다.

```text
LLM의 기본 원리:
P(next_token | previous_tokens)
→ 이전 토큰들이 주어졌을 때, 다음 토큰의 확률 분포

예:
입력: "안녕하세요, 제 이름은"
LLM: "Claude입니다" (가장 확률 높은 다음 토큰)
입력: "안녕하세요, 제 이름은 Claude입니다. 저는"
LLM: "AI 어시스턴트입니다" (맥락 고려)
```

### LLM의 용도

```text
- 질의응답 (Q&A)
- 텍스트 요약
- 코드 작성 및 설명
- 창작 글쓰기
- 추론 및 분석
- 번역
- 대화
```

---

## Token

### "token은 단어와 무엇이 다른가?"

**Token은 단어보다 작은 단위 (부분 문자, 단어, 구절)로 정확하게 어떻게 분할하느냐에 따라 비용과 성능이 결정**된다.

```text
문장: "Hello, how are you?"

단어 기준:
["Hello", ",", "how", "are", "you", "?"] → 6개

Token 기준 (GPT-2):
["Hello", ",", " how", " are", " you", "?"] → 6개

Token 기준 (더 분리):
["H", "e", "l", "l", "o", ",", " how", " are", " you", "?"] → 10개

한국어:
"안녕하세요, 이름이 뭐예요?"

단어 기준: ["안녕하세요", ",", "이름", "이", "뭐", "예요", "?"] → 7개

Token 기준: 더 작게 분할 가능 → 10개 이상
```

### Tokenization의 영향

```text
Token이 많을수록:
✗ 비용 증가 (token 기반 과금)
✓ 정확도 증가 (세부 정보 보존)

Token이 적을수록:
✓ 비용 절감
✗ 정보 손실 (문자 손실)

한국어/중국어:
- 공백 없음
- Token 많아짐
- 비용 높음
- 영어 1.5~2배 비용
```

---

## Context Window

### "context window는 어떤 제약을 만드는가?"

**Context Window는 모델이 한번에 처리할 수 있는 최대 토큰 개수로, 이를 초과하면 오래된 정보를 잃는다.**

```text
Context Window: 4096 tokens (예)

문장 1 (500 tokens)
문장 2 (500 tokens)
문장 3 (500 tokens)
문장 4 (500 tokens)
→ 총 2000 tokens (OK, window 내)

문장 5 (3000 tokens 추가)
→ 총 5000 tokens (window 초과!)
→ 문장 1 일부가 삭제됨 (오래된 정보 손실)

결과:
모델은 문장 1을 기억하지 못함
→ 일관성 있는 응답 어려움
```

### 제약 극복

```text
1. 더 큰 Context Window
   - GPT-3: 4K
   - GPT-4: 8K, 32K, 128K
   - Claude: 100K

2. 요약 (Summarization)
   - 긴 문서 먼저 요약
   - 요약본을 context에 포함

3. 검색 (RAG: Retrieval Augmented Generation)
   - 관련 정보만 context에 포함
   - 전체 문서 저장 불필요
```

---

## Pretraining

### Pretraining이란?

```text
목표: 일반적인 언어 패턴 학습

과정:
대규모 텍스트 코퍼스
    ↓ (수조 tokens)
무감독 학습
- 다음 token 예측 (Causal LM)
- Masked token 예측 (MLM)
    ↓
일반적 언어 모델
(GPT-3, LLaMA, PaLM)
```

### 비용

```text
Pretraining:
- 시간: 몇 주~몇 개월
- GPU: 수천 개
- 비용: 수백만 달러
- 회사: 매우 큰 기업만 가능

Fine-tuning:
- 시간: 몇 시간~며칠
- GPU: 수십 개
- 비용: 수천~수만 달러
- 개인: 가능
```

---

## Instruction Tuning

### Instruction Tuning이란?

```text
Pretraining 후:
모델은 문장 완성만 가능
"Question: 2+2 =
Answer:" (완성 예상: 4? 5?)

Instruction Tuning 후:
모델은 지시를 따름
"2+2는 몇인가?" → "4입니다"

과정:
(question, answer) 쌍으로 미세 조정
→ 지시 이해 능력 향상
```

### 과정

```text
1. 데이터 수집
   (질문, 답변) 쌍
   예: ("What is 2+2?", "4")

2. 미세 조정
   Pretraining된 모델에서 시작
   위 쌍으로 학습

3. 결과
   지시를 따르는 모델
```

---

## Inference

### Inference 과정

```text
입력: "안녕하세요"

1. Tokenization
   ["안", "녕", "하", "세", "요"] → [토큰 ID]

2. Model Forward Pass
   토큰 ID → Embedding → Transformer → Logits

3. Next Token 생성
   Logits → 확률 분포 → Sampling/Greedy

4. Decoding
   토큰 ID → 텍스트 "반갑습니다"

5. 반복
   이전 토큰들 + 생성된 토큰으로 다음 토큰 생성
```

### Sampling vs Greedy

```text
Greedy (결정적):
- 가장 확률 높은 토큰 선택
- 결과: 항상 같음
- 예: "안녕" → 항상 "하세요"

Sampling (무작위):
- 확률 분포에서 샘플링
- 결과: 매번 다름
- 예: "안녕" → "하세요" 또는 "하십니까"

실무:
- 창작: Sampling (다양성)
- 정보성: Greedy (일관성)
```

---

## "temperature는 출력에 어떤 영향을 주는가?"

**Temperature는 모델의 "창의성" 수준을 조절한다. 높을수록 다양하고 창의롭지만 부정확할 수 있고, 낮을수록 일관되지만 반복될 수 있다.**

```text
Temperature 스케일:
0: 결정적 (greedy 선택)
0.3: 보수적 (Q&A, 정보성)
1.0: 기본값
1.5: 창의적 (창작, 대화)
2.0: 무작위에 가까움

예:
Temperature 0 (확률 분포):
[0.7, 0.2, 0.1] → 최대값 0.7만 선택

Temperature 1 (기본):
[0.7, 0.2, 0.1] → 그대로 사용
0.7 확률로 첫 토큰, 0.2 확률로 두 번째...

Temperature 2 (높음):
확률 분포 "평탄화"
[0.6, 0.3, 0.1] → [0.5, 0.4, 0.1] (더 균등)
→ 낮은 확률 토큰도 선택 가능
```

---

## Hallucination

### "hallucination은 왜 발생하는가?"

**Hallucination은 모델이 학습 데이터에 없는 정보를 그럴듯하게 만들어내는 현상**이다. 언어 모델은 "다음 토큰 확률"만 학습했지 "사실성"은 모른다.

```text
실제 사건:
"2020년 미국 대선: Biden이 Trump 이김"

Hallucination 예:
Q: "2020년 미국 대선 후보는?"
A: "Biden, Trump, Kennedy, ...
   Kennedy는 실제 후보가 아님!
   하지만 그럴듯한 이름이라 생성"

원인:
- 학습 데이터에 없는 정보
- 패턴 일반화 과정에서 오류
- "확률 높은 다음 토큰" ≠ "사실"
```

### 방지 방법

```text
1. RAG (Retrieval Augmented Generation)
   - 사실 정보를 외부에서 검색
   - 검색 결과를 context에 포함
   - 모델: 검색된 정보 기반으로 생성

2. Fine-tuning
   - 신뢰할 수 있는 데이터로 미세 조정
   - RLHF (Reinforcement Learning from Human Feedback)

3. Fact Checking
   - 모델 출력을 검증
   - 신뢰도 점수 제공

4. Temperature 낮추기
   - 낮은 확률 토큰 선택 감소
   - 하지만 완전 방지 불가
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Token | 텍스트의 작은 단위 (단어보다 작음) |
| Context Window | 한번에 처리 가능한 최대 토큰 수 |
| Pretraining | 대규모 텍스트에서 일반 패턴 학습 |
| Instruction Tuning | 지시 따르기 능력 학습 |
| Inference | 학습된 모델로 텍스트 생성 |
| Hallucination | 그럴듯한 거짓 정보 생성 |

---

## Related Notes

- [Prompt Engineering](../prompt-engineering/overview.md)
- [RAG](../rag/overview.md)
