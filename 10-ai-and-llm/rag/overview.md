# RAG

## 목차
- [RAG의 역할](#rag의-역할)
- [Retrieval](#retrieval)
- [Embedding](#embedding)
- [Vector Search](#vector-search)
- [Context Injection](#context-injection)
- [Grounding](#grounding)
- [Evaluation](#evaluation)

---

## RAG의 역할

**RAG (Retrieval Augmented Generation)는 LLM이 답변하기 전에 외부 지식 저장소에서 관련 문서를 검색해 프롬프트에 주입하는 기법**이다. 할루시네이션을 줄이고 최신 정보를 활용할 수 있다.

```text
RAG 없이 (Parametric Knowledge):
Q: "회사 정책은?"
A: "LLM 학습 데이터에만 기반
→ 오래된 정보, 할루시네이션 위험"

RAG 있이 (Retrieved Knowledge):
Q: "회사 정책은?"
→ 사내 정책 문서 검색
→ 최신 정책 문단을 prompt에 포함
A: "최신 정책에 기반한 정확한 답변"
```

### RAG vs Fine-tuning

**RAG는 동적 지식 추가(검색), Fine-tuning은 정적 지식 추가(재학습)**이다.

```text
RAG (Retrieval Augmented Generation):
- 방식: 질문 시 외부 문서 검색
- 시간: 즉시 (밀리초)
- 비용: 저렴 (검색만)
- 업데이트: 즉시 (문서 추가하면 바로 반영)
- 사용 시점: 자주 바뀌는 정보 (뉴스, 정책)

Fine-tuning:
- 방식: 모델 학습
- 시간: 오래 걸림 (몇 시간~몇 일)
- 비용: 비쌈 (GPU, 학습)
- 업데이트: 느림 (재학습 필요)
- 사용 시점: 기본 스타일/능력 조정 (전문 용어, 톤)
```

### RAG 아키텍처

```text
1. Indexing (오프라인)
   문서 수집
       ↓
   청킹 (Chunking)
       ↓
   Embedding 변환
       ↓
   Vector DB 저장

2. Retrieval (온라인)
   사용자 질문
       ↓
   질문 Embedding
       ↓
   Vector Search
       ↓
   상위 K개 문서 선택

3. Generation
   검색된 문서 + 질문
       ↓
   프롬프트 구성
       ↓
   LLM 생성
```

---

## Retrieval

### "Retrieval은 어떤 방식으로 동작하는가?"

**Retrieval은 사용자 질문과 유사한 문서를 저장소에서 찾아내는 과정**이다. 키워드 검색과 의미 기반 검색이 있다.

```text
키워드 검색 (Keyword Search):
질문: "Python 설치 방법"
→ "Python", "설치" 포함 문서 검색
→ 단어 매칭 (TF-IDF)
장점: 빠르고 정확
단점: 동의어, 의미 변형 못 감지

의미 기반 검색 (Semantic Search):
질문: "Python 설치 방법"
→ 문장 의미를 벡터로 변환
→ 벡터 유사도로 검색
→ "파이썬 세팅", "Python 구성" 등도 찾음
장점: 의미 이해
단점: 느림, 모델 의존
```

### 하이브리드 검색

```text
장점만 결합:
1. 키워드 검색으로 후보 선별
2. 벡터 검색으로 순서 정렬
3. 두 결과 통합

예:
질문: "Apache Kafka의 성능 최적화"
키워드: Apache, Kafka, 성능, 최적화
벡터 유사도: "메시지 큐 처리량" 도 포함
```

### "chunk size는 검색 품질에 어떤 영향을 주는가?"

**청크 크기는 검색의 정확도와 맥락의 완전성 사이의 균형**이다.

```text
청크 너무 작음 (예: 50단어):
장점: 정확한 결과
단점: 맥락 부족
→ "이 문장만으로는 이해 어려움"

청크 적당함 (예: 300단어):
장점: 맥락 충분, 정확도 높음
단점: 없음

청크 너무 큼 (예: 2000단어):
장점: 완전한 맥락
단점: 노이즈 많음, 검색 부정확
→ "너무 많은 정보, 관련 없는 내용 포함"
```

### 청크 크기 가이드

```text
일반 문서: 300-500 단어 (약 1500-2500 글자)
코드 문서: 200-300 단어 (함수 하나 정도)
법률 문서: 500-800 단어 (조항 단위)

오버래핑:
청크 간 겹침 (50-100 단어)
→ 경계 부분에서 맥락 손실 방지
```

---

## Embedding

### "embedding model은 왜 중요한가?"

**Embedding Model은 텍스트를 벡터로 변환하는 핵심 요소로, 모델 품질이 검색 성능을 좌우**한다.

```text
나쁜 Embedding:
- "강아지"와 "개"를 다른 벡터로 변환
- 의미 유사성 못 포착
→ 검색 실패

좋은 Embedding:
- "강아지"와 "개"를 비슷한 벡터로 변환
- 의미 관계 반영
→ 정확한 검색
```

### 인기 있는 Embedding 모델

```text
OpenAI text-embedding-3:
- 가격: 유료
- 성능: 최고 수준
- 차원: 1536 또는 256

Sentence Transformers (오픈소스):
- 가격: 무료
- 성능: 좋음
- 차원: 384-768
- 예: all-MiniLM-L6-v2

Cohere Embed:
- 가격: 유료 (저렴)
- 성능: 좋음
- 특징: 동적 차원 조정

기준 선택:
- 비용 민감 → 오픈소스
- 성능 중시 → OpenAI, Cohere
- 특화 도메인 → Fine-tuned 모델
```

### Embedding 구현 예시

```python
from sentence_transformers import SentenceTransformer

# 모델 로드
model = SentenceTransformer('all-MiniLM-L6-v2')

# 문서 Embedding
documents = [
    "Python은 프로그래밍 언어다",
    "Java도 프로그래밍 언어다",
    "고양이는 동물이다"
]

embeddings = model.encode(documents)
# embeddings[0]: [0.1, -0.2, 0.3, ...] (384 차원)

# 쿼리 Embedding
query = "프로그래밍 언어"
query_embedding = model.encode(query)

# 유사도 계산 (코사인 거리)
from sklearn.metrics.pairwise import cosine_similarity
scores = cosine_similarity([query_embedding], embeddings)
# scores[0]: [0.95, 0.92, 0.15] (높을수록 유사)
```

---

## Vector Search

### "Vector Search는 어떻게 효율적으로 동작하는가?"

**Vector Search는 고차원 벡터에서 유사한 벡터를 빠르게 찾는 방식**이다. 정확한 매칭이 아니라 근사(approximate) 검색을 사용한다.

```text
정확 검색 (Brute Force):
모든 벡터와 비교 (O(n))
→ 정확하지만 느림 (백만개 데이터: 초 단위)

근사 검색 (ANN: Approximate Nearest Neighbor):
Indexing 활용 (O(log n))
→ 빠르지만 약간 부정확 (백만개 데이터: 밀리초)
```

### Vector DB (벡터 데이터베이스)

```text
Pinecone:
- SaaS (서버 관리 필요 없음)
- 가격: 비쌈
- 특징: 서버리스, 스케일링 쉬움

Weaviate:
- 오픈소스 + 클라우드
- 가격: 무료/유료
- 특징: GraphQL API, 하이브리드 검색

Chroma:
- 오픈소스, 경량
- 가격: 무료
- 특징: 임베드 가능, 개발용

Milvus:
- 오픈소스, 대규모
- 가격: 무료
- 특징: 고성능, 엔터프라이즈
```

### Vector Search 예시

```python
# Pinecone 예시
import pinecone

# 초기화
pinecone.init(api_key="xxx", environment="us-west1-gcp")
index = pinecone.Index("documents")

# Embedding 저장
embeddings = [
    ("doc1", [0.1, -0.2, 0.3, ...]),
    ("doc2", [0.15, -0.18, 0.32, ...])
]
index.upsert(vectors=embeddings)

# 검색
query_embedding = [0.12, -0.19, 0.31, ...]
results = index.query(query_embedding, top_k=5)
# results: [("doc1", 0.98), ("doc2", 0.95), ...]
```

---

## Context Injection

### "검색된 문서를 프롬프트에 어떻게 삽입할 것인가?"

**Context Injection은 검색 결과를 LLM 프롬프트에 구조화된 방식으로 포함**하는 방법이다.

```text
나쁜 삽입:
Q: "회사 휴가 정책?"
Context 포함한 Prompt:
"여기 무작위 문서들이 있어. 답변해.
[문서1: PDF 원본 그대로]
[문서2: 관계없는 텍스트]
..."
→ 노이즈 많음

좋은 삽입:
Q: "회사 휴가 정책?"
Context 포함한 Prompt:
"다음은 회사 정책 문서 발췌다:

## 휴가 정책
- 연차: 15일
- 사용 방법: 인사팀 신청
- 기한: 연말까지 미사용 폐기

이를 기반으로 질문에 답해."
→ 명확하고 구조화됨
```

### 효과적 Context 구성

```text
1. 명확한 구분
   "다음은 검색된 문서다:"
   "---"
   [검색 결과]
   "---"

2. 출처 표시
   "[문서: policy.md]"
   "[페이지: 3]"
   → 할루시네이션 추적 용이

3. 우선순위 정렬
   가장 관련 높은 것부터
   → 토큰 제한 시 상위 것부터 포함

4. 길이 제한
   각 문서 200-300 단어로 자르기
   → Context window 효율성
```

### 구현 예시

```python
def build_rag_prompt(query, retrieved_docs):
    context = ""
    for i, doc in enumerate(retrieved_docs, 1):
        context += f"[문서 {i}: {doc['source']}]\n"
        context += f"{doc['content'][:300]}\n\n"  # 300단어 자르기
    
    prompt = f"""다음은 관련 문서 발췌다:

{context}

---

사용자 질문: {query}

위 문서를 기반으로 정확하게 답변해."""
    
    return prompt
```

---

## Grounding

### "Grounding은 왜 중요한가?"

**Grounding은 LLM의 답변이 검색된 문서에 기반하도록 강제해 할루시네이션을 줄인다.**

```text
Grounding 없이:
Q: "회사 정책은?"
A: "회사는 유연한 근무를 지원합니다
    (학습 데이터 기반, 틀릴 가능성)"

Grounding 있이:
Q: "회사 정책은?"
A: "[문서: policy.md에 따르면]
    회사는 주 3일 재택근무를 지원합니다"
    (검색된 문서 명시)
```

### Grounding 구현

```text
1. 명시적 출처 요구
   Prompt: "답변할 때 출처를 명시해"
   A: "[HR 정책 v2.1] 유연근무..."

2. Citation 강제
   Prompt: "답변을 [문서 1, 문서 2]로 뒷받침해"
   A: "[1] 첫 번째 정보 [2] 두 번째 정보"

3. 모른다 강제
   Prompt: "문서에 없으면 '모른다'고 말해"
   A: "문서에 명시되지 않아 알 수 없습니다"
```

### "retrieved context가 틀리면 답변은 어떻게 망가지는가?"

**검색 결과가 부정확하면 LLM은 이를 신뢰하고 부정확한 답변을 생성한다. 이것이 RAG의 약점**이다.

```text
상황: 가격 정보 검색
검색 결과 (잘못됨):
"제품 A 가격: $100"
(실제: $200)

LLM 답변:
"제품 A는 $100입니다" (잘못된 정보 전파)

예방책:
1. 검색 정확도 향상
   - 더 나은 Embedding 모델
   - 더 나은 청크 크기
   
2. 재검증
   - 중요한 정보는 별도 검증
   - 출처 확인
   
3. Prompt 강화
   - "정보가 불확실하면 표시해"
   - "모순을 감지하면 언급해"
```

---

## Evaluation

### "RAG 성능을 어떻게 평가할 것인가?"

**RAG 평가는 Retrieval 품질과 Generation 품질을 분리해서 측정**해야 한다.

```text
전체 평가만으로는:
Q: "A는?"
A: "B입니다" (틀렸음)

→ 검색 잘못? 생성 잘못? 알 수 없음

분리 평가:
1. Retrieval 평가: 관련 문서가 검색되었나?
2. Generation 평가: 검색된 문서를 잘 활용했나?
```

### Retrieval 평가

```text
Recall@K:
상위 K개 중 관련 문서의 비율
→ K=5일 때 5개 중 몇 개가 관련있나?

NDCG (Normalized Discounted Cumulative Gain):
순서를 고려한 평가
→ 1위가 틀리면 더 큰 페널티

MRR (Mean Reciprocal Rank):
첫 정답 위치의 역수 평균
→ 첫 정답이 3위면 1/3
```

### Generation 평가

```text
BLEU Score:
생성 텍스트와 정답의 n-gram 일치도
(기계 번역용, RAG에서는 덜 적합)

ROUGE Score:
회수 기반 평가
→ 요약 작업에 유용

Human Evaluation:
1. 정확도: 정보가 맞나?
2. 완전성: 질문을 모두 답했나?
3. 신뢰성: 출처가 명확한가?
4. 가독성: 이해하기 쉬운가?
```

### 평가 자동화 예시

```python
def evaluate_rag(questions, retrieved_docs, answers):
    results = {
        'retrieval': [],
        'generation': []
    }
    
    # Retrieval 평가
    for q, docs in zip(questions, retrieved_docs):
        relevant_count = sum(1 for d in docs if is_relevant(q, d))
        recall = relevant_count / len(docs)
        results['retrieval'].append(recall)
    
    # Generation 평가
    for ans, true_ans in zip(answers, true_answers):
        bleu = calculate_bleu(ans, true_ans)
        rouge = calculate_rouge(ans, true_ans)
        results['generation'].append({'bleu': bleu, 'rouge': rouge})
    
    return results
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Retrieval | 질문과 유사한 문서 검색 |
| Embedding | 텍스트를 벡터로 변환 |
| Vector Search | 벡터 유사도로 검색 |
| Context Injection | 검색 결과를 프롬프트에 삽입 |
| Grounding | 답변을 검색 문서에 기반하게 |
| Evaluation | RAG 성능 평가 |

---

## Related Notes

- [LLM](../llm/overview.md)
- [Vector Database](../vector-database/overview.md)
