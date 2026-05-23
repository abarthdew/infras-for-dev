# Vector Database

## 목차
- [Vector Database의 역할](#vector-database의-역할)
- [Embedding Vector](#embedding-vector)
- [Similarity Search](#similarity-search)
- [Index](#index)
- [Metadata Filtering](#metadata-filtering)
- [Recall과 Latency](#recall과-latency)

---

## Vector Database의 역할

**Vector Database는 텍스트, 이미지, 음성 등을 숫자 벡터로 저장하고 의미적으로 유사한 항목을 빠르게 검색하는 데 특화된 저장소**다. 전통 데이터베이스(SQL)와 다르게 의미 기반 검색에 최적화되어 있다.

```text
전통 데이터베이스 (정확 매칭):
SELECT * FROM users WHERE age = 25
→ 정확히 25세만 찾음

벡터 데이터베이스 (의미 유사도):
SELECT * FROM documents WHERE embedding SIMILAR_TO query_embedding
→ 의미가 비슷한 모든 문서 찾음
```

### 사용 사례

```text
1. RAG (Retrieval Augmented Generation)
   Q: "Python 설치 방법"
   → 비슷한 문서 검색
   → LLM에 주입

2. 추천 시스템
   사용자 A의 취향 벡터
   → 비슷한 취향의 사용자 B 찾기
   → 비슷한 상품 추천

3. 이미지 검색
   사진 벡터
   → 비슷한 사진 검색 (고양이 사진 찾기)

4. 이상 탐지
   정상 거래 벡터
   → 비슷하지 않은 거래 탐지
```

### 벡터 DB vs SQL DB

```text
SQL Database:
- 데이터: 구조화된 행과 열
- 검색: 정확 매칭, 범위 쿼리
- 인덱스: B-Tree
- 예: PostgreSQL, MySQL

Vector Database:
- 데이터: 고차원 벡터
- 검색: 유사도 기반
- 인덱스: LSH, HNSW, IVF
- 예: Pinecone, Weaviate, Milvus
```

---

## Embedding Vector

### "embedding은 무엇을 숫자로 표현하는가?"

**Embedding은 텍스트, 이미지 등을 의미를 담은 고차원 숫자 벡터로 변환한 것**이다. 의미적으로 비슷한 것은 벡터 공간에서도 가까워진다.

```text
텍스트: "좋은 영화야"
↓ Embedding Model
벡터: [0.23, -0.15, 0.88, ..., 0.42]
(384차원 또는 1536차원)

의미 관계:
"좋은 영화" 벡터 ≈ "훌륭한 영화" 벡터
"나쁜 영화" 벡터는 멀음
```

### Embedding의 성질

```text
1. 의미 보존
   - 비슷한 의미 → 비슷한 벡터
   - 반대 의미 → 멀은 벡터

2. 방향성
   - 벡터의 방향이 의미를 담음
   - 크기는 의미와 무관

3. 대수 연산
   king - man + woman ≈ queen
   (개념적 유사성도 대수적으로 표현)

예:
"서울" - "한국" + "일본" ≈ "도쿄"
"좋음" + "상품" ≈ "우수한 제품"
```

### Embedding 생성

```python
from sentence_transformers import SentenceTransformer

# 모델 로드 (384차원)
model = SentenceTransformer('all-MiniLM-L6-v2')

# 단일 문자열
text = "Python 프로그래밍"
embedding = model.encode(text)
print(embedding.shape)  # (384,)
print(embedding[:5])    # [0.12, -0.34, 0.56, ...]

# 배치 처리
texts = [
    "Python 프로그래밍",
    "자바 개발",
    "강아지는 귀엽다"
]
embeddings = model.encode(texts)
print(embeddings.shape)  # (3, 384)
```

### 차원의 의미

```text
차원: 벡터의 길이
- 작은 차원 (256, 384): 빠르고 저렴
  → RAG, 검색
  
- 큰 차원 (1536, 3072): 정확하지만 느림
  → 이미지, 복잡한 의미 포착
  
절충:
대부분의 RAG: 384-768차원으로 충분
```

---

## Similarity Search

### "cosine similarity는 무엇을 비교하는가?"

**Cosine Similarity는 두 벡터가 같은 방향을 얼마나 가리키는지 비교**한다. 벡터 공간에서 각도를 측정한다.

```text
벡터 A: [1, 0, 0]
벡터 B: [1, 0.1, 0]
벡터 C: [0, 1, 0]

코사인 유사도:
sim(A, B) = 0.99 (거의 같은 방향)
sim(A, C) = 0 (수직, 전혀 다른 방향)
```

### 유사도 계산

```text
코사인 유사도 공식:
similarity = (A · B) / (||A|| × ||B||)

범위: -1 ~ 1 (보통 0 ~ 1)
- 1: 같은 방향
- 0: 수직 (관계없음)
- -1: 반대 방향

벡터 A = [3, 4]
벡터 B = [5, 12]

내적: 3×5 + 4×12 = 63
||A||: √(9+16) = 5
||B||: √(25+144) = 13

sim(A,B) = 63 / (5×13) = 0.97
```

### 다른 유사도 지표

```text
L2 거리 (Euclidean Distance):
- 두 점 사이 직선 거리
- 차원 증가하면 신뢰도 떨어짐
- 사용: 이미지, 비디오

Dot Product:
- 내적 직접 사용
- 빠르지만 벡터 크기 영향받음
- 사용: 대규모 검색

Manhattan Distance:
- 격자 거리
- 계산 빠르지만 정확도 낮음
```

### 구현 예시

```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

# 벡터들
query = np.array([[0.1, 0.5, 0.3]])  # (1, 3)
documents = np.array([
    [0.12, 0.48, 0.32],  # 유사
    [0.9, 0.1, 0.1],     # 다름
    [0.11, 0.51, 0.29]   # 유사
])

# 유사도 계산
similarities = cosine_similarity(query, documents)[0]
# [0.99, 0.12, 0.98]

# 순서대로 정렬
sorted_indices = np.argsort(-similarities)
# [0, 2, 1]
```

---

## Index

### "Index는 무엇을 위해 필요한가?"

**Index는 백만 개 이상의 벡터를 빠르게 검색하기 위한 구조**다. 모든 벡터와 비교하면 시간이 오래 걸리므로 인덱싱이 필수다.

```text
인덱스 없이 (Brute Force):
100만 개 벡터 × 쿼리 벡터 비교
→ 모든 내적 계산
→ 시간: ~초 단위 (느림)

인덱스 있이 (ANN):
클러스터링 활용
→ 관련 영역만 검색
→ 시간: ~밀리초 (빠름)
```

### "ANN index는 왜 정확도와 속도 trade-off가 있는가?"

**ANN (Approximate Nearest Neighbor)은 정확한 최근접을 보장하지 않고 근사값을 빠르게 찾기 때문에 속도를 얻는 대신 정확도를 포기**한다.

```text
정확 검색 (Exact NN):
- 방식: 모든 벡터와 비교
- 정확도: 100%
- 속도: 느림 (초 단위)

근사 검색 (ANN):
- 방식: 인덱싱으로 효율화
- 정확도: ~95% (조정 가능)
- 속도: 빠름 (밀리초)
```

### 인기 있는 인덱스

```text
HNSW (Hierarchical Navigable Small World):
- 계층적 구조
- 속도: 빠름
- 정확도: 높음
- 메모리: 중간
- 사용: Pinecone, Weaviate

IVF (Inverted File):
- 클러스터링 기반
- 속도: 매우 빠름
- 정확도: 중간
- 메모리: 적음
- 사용: FAISS (Facebook)

LSH (Locality Sensitive Hashing):
- 해시 기반
- 속도: 빠름
- 정확도: 낮음
- 메모리: 적음

FAISS (Facebook AI Similarity Search):
- 프레임워크
- 여러 인덱스 방식 지원
- 오픈소스, 고성능
```

### 인덱스 구성

```text
Partition Phase:
전체 벡터 공간을 여러 영역으로 분할
→ 각 영역에 대표 점(centroid) 지정

Search Phase:
쿼리 벡터와 가장 가까운 centroid 찾기
→ 그 영역의 벡터들만 상세 검색
```

### 구현 예시 (FAISS)

```python
import faiss
import numpy as np

# 벡터 준비
vectors = np.random.random((100000, 384)).astype('float32')

# HNSW 인덱스 생성
index = faiss.IndexHNSWFlat(384, 32)
index.add(vectors)

# 쿼리
query = np.random.random((1, 384)).astype('float32')
distances, indices = index.search(query, k=5)
# indices: [[0, 234, 5678, ...]]  (상위 5개)
```

---

## Metadata Filtering

### "metadata filter는 언제 필요한가?"

**Metadata Filtering은 벡터 검색 결과를 속성(메타데이터)로 추가 필터링해 더 정확한 결과를 얻을 때 사용**한다.

```text
필터 없이:
Q: "좋은 영화"
→ 검색: [영화1(2020), 영화2(2023), 영화3(2010)]
(연도 상관없음)

필터 있이:
Q: "최근 5년의 좋은 영화"
Metadata: year >= 2021
→ 검색 후 필터: [영화2(2023)]
(최신 영화만)
```

### 메타데이터 예시

```text
문서 청크:
{
  "id": "doc123_chunk5",
  "text": "Python은 ...",
  "embedding": [0.1, -0.2, ...],
  "metadata": {
    "source": "tutorial.md",
    "version": "2.1",
    "date": "2026-05-23",
    "category": "programming",
    "difficulty": "beginner"
  }
}
```

### 메타데이터 필터링

```text
필터 조건:
- 동등: category == "programming"
- 범위: difficulty >= "intermediate"
- 목록: source IN ["guide.md", "faq.md"]
- 범위: date BETWEEN "2024-01-01" AND "2026-12-31"

조합:
category == "programming" AND difficulty >= "beginner"
AND date >= "2025-01-01"
```

### 구현 예시 (Pinecone)

```python
import pinecone

# 초기화
pinecone.init(api_key="xxx")
index = pinecone.Index("documents")

# 메타데이터와 함께 저장
vectors_with_metadata = [
    ("id1", [0.1, 0.2, ...], {"category": "python", "year": 2025}),
    ("id2", [0.15, 0.25, ...], {"category": "java", "year": 2024}),
]
index.upsert(vectors=vectors_with_metadata)

# 필터링 검색
query_embedding = [0.12, 0.22, ...]
results = index.query(
    vector=query_embedding,
    top_k=5,
    filter={"year": {"$gte": 2024}}  # 2024년 이후만
)
# [('id1', 0.98), ...] (필터 조건 만족)
```

---

## Recall과 Latency

### "성능 지표 Recall과 Latency의 의미"

**Recall은 검색 정확도, Latency는 검색 속도를 나타낸다. RAG에서는 두 지표 모두 중요하고 균형을 맞춰야 한다.**

```text
Recall:
- 정의: "실제 최근접 이웃 중 찾은 비율"
- 범위: 0~1 (높을수록 좋음)
- 예: Recall@5 = 0.95
  (상위 5개 중 95%가 정답)

Latency:
- 정의: "검색 요청부터 응답까지의 시간"
- 범위: 밀리초
- 예: 10ms (RAG: 50-100ms 목표)
```

### Trade-off

```text
높은 Recall (정확도 우선):
- 방식: Brute Force 또는 조건 완화
- Recall: 99%
- Latency: 1000ms (느림)
- 사용: 오프라인 분석

높은 속도 (Latency 우선):
- 방식: 빠른 인덱싱 (IVF 적극 활용)
- Recall: 80%
- Latency: 10ms (빠름)
- 사용: 실시간 검색

균형 (일반적):
- 방식: HNSW, 메타데이터 필터링
- Recall: 95%
- Latency: 50ms
- 사용: RAG, 추천 시스템
```

### 최적화 전략

```text
1. 인덱스 매개변수 조정
   - HNSW의 ef (expansion factor) 증가 → 정확도 ↑, 속도 ↓
   - IVF의 nprobe (탐색 프로브) 증가 → 정확도 ↑, 속도 ↓

2. Quantization (양자화)
   - 벡터 압축 (float32 → int8)
   - 메모리 90% 감소, 속도 향상
   - 정확도 약간 손실

3. Caching
   - 자주 검색되는 쿼리 캐싱
   - Latency 대폭 감소

4. 배치 처리
   - 여러 쿼리를 한 번에 처리
   - 처리량 향상
```

### 성능 측정 예시

```python
import time
from sklearn.metrics import recall_score

def evaluate_vector_db(vectors, queries, top_k=5):
    start_time = time.time()
    
    # 검색 (HNSW)
    distances, indices = index.search(queries, k=top_k)
    
    # 속도 측정
    latency = (time.time() - start_time) / len(queries)  # 평균 ms
    
    # 정확도 측정 (정답과 비교)
    found = sum(1 for idx in indices if idx in ground_truth)
    recall = found / (len(queries) * top_k)
    
    print(f"Recall@{top_k}: {recall:.2%}")
    print(f"Avg Latency: {latency*1000:.1f}ms")
    
    return {"recall": recall, "latency": latency}
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Embedding | 의미를 담은 고차원 벡터 |
| Similarity | 벡터 간 유사도 측정 |
| Cosine | 각도 기반 유사도 |
| Index | 빠른 검색을 위한 자료 구조 |
| ANN | 근사 최근접 탐색 |
| Metadata | 벡터 추가 속성 (필터링용) |
| Recall | 검색 정확도 |
| Latency | 검색 속도 |

---

## Related Notes

- [LLM](../llm/overview.md)
- [RAG](../rag/overview.md)
