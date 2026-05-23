# Project: RAG Demo

## 목차
- [프로젝트 목표](#프로젝트-목표)
- [문서 수집](#문서-수집)
- [Chunking](#chunking)
- [Embedding](#embedding)
- [Retrieval](#retrieval)
- [Answer Generation](#answer-generation)
- [구현 예시](#구현-예시)
- [평가 및 최적화](#평가-및-최적화)

---

## 프로젝트 목표

**문서를 청크로 분할하고 Embedding해서 벡터 DB에 저장한 후, 사용자 질문에 관련된 문서를 검색하여 LLM으로 답변을 생성하는 RAG 파이프라인을 구현**한다.

```text
RAG Pipeline:
문서 수집
    ↓
청킹 (Chunking)
    ↓
Embedding 생성
    ↓
벡터 DB 저장
    ↓
(온라인) 질문 입력
    ↓
쿼리 Embedding
    ↓
유사 문서 검색
    ↓
문서 + 질문 → LLM
    ↓
최종 답변 + 출처
```

---

## 문서 수집

### 문서 소스

```python
from pathlib import Path
import os

# 문서 소스 다양성
sources = [
    # 1. 로컬 파일
    ("markdown", Path("docs/*.md")),
    
    # 2. 웹 페이지
    ("url", "https://example.com/docs"),
    
    # 3. PDF
    ("pdf", "documents/guide.pdf"),
    
    # 4. 데이터베이스
    ("sql", "SELECT content FROM articles")
]

# 간단한 Markdown 문서로 시작
documents = []
for md_file in Path("docs").glob("*.md"):
    with open(md_file, "r", encoding="utf-8") as f:
        content = f.read()
        documents.append({
            "source": md_file.name,
            "content": content,
            "type": "markdown"
        })
```

### 문서 예시

```markdown
# NAT (Network Address Translation)

## 정의
NAT는 한 개의 공인 IP 주소로 여러 개의 비공개 IP 주소를 매핑하는 기술이다.

## 동작 방식
1. 내부 호스트는 비공개 IP (192.168.1.10)을 사용
2. 라우터는 비공개 IP를 공인 IP로 변환
3. 외부와 통신

## 종류
- Static NAT: 1:1 매핑
- Dynamic NAT: 여러 호스트 공유
- PAT (Port Address Translation): 포트까지 변환
```

---

## Chunking

### "chunk 크기는 검색 품질에 어떤 영향을 주는가?"

**청크 크기는 검색 정확도와 맥락의 완전성 사이의 균형을 결정**한다.

```text
청크 너무 작음 (예: 50단어):
✓ 정확한 검색
✗ 맥락 부족 ("이 문장만으로는 이해 안 됨")

청크 적당함 (예: 300-500단어):
✓ 정확한 검색
✓ 충분한 맥락

청크 너무 큼 (예: 2000단어):
✓ 완전한 맥락
✗ 노이즈 많음 (관련 없는 정보)
✗ 검색 부정확
```

### Chunking 전략

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 재귀적 분할
splitter = RecursiveCharacterTextSplitter(
    chunk_size=300,          # 300단어
    chunk_overlap=50,        # 50단어 겹침
    separators=["\n\n", "\n", ".", " "]  # 분할 우선순위
)

documents = [
    {
        "source": "nat.md",
        "content": "NAT는... (긴 문서)"
    }
]

chunks = []
for doc in documents:
    splits = splitter.split_text(doc["content"])
    for i, chunk_text in enumerate(splits):
        chunks.append({
            "source": doc["source"],
            "chunk_id": f"{doc['source']}_chunk_{i}",
            "text": chunk_text,
            "order": i
        })
```

### 겹침(Overlap)의 중요성

```text
겹침 없이:
Chunk 1: "NAT는 한 개의 공인 IP로 여러 개의..."
Chunk 2: "...비공개 IP를 매핑하는 기술이다. 동작 방식은..."
경계에서 의미 손실

겹침 있이:
Chunk 1: "NAT는 한 개의 공인 IP로 여러 개의 비공개 IP를 매핑하는 기술이다."
Chunk 2: "...비공개 IP를 매핑하는 기술이다. 동작 방식은..."
경계 부분 중복으로 맥락 유지
```

### Chunk 메타데이터

```python
chunks_with_metadata = []
for chunk in chunks:
    chunks_with_metadata.append({
        "id": chunk["chunk_id"],
        "text": chunk["text"],
        "metadata": {
            "source": chunk["source"],
            "chunk_order": chunk["order"],
            "token_count": len(chunk["text"].split()),
            "date": "2026-05-23"
        }
    })
```

---

## Embedding

### 임베딩 모델 선택

```python
from sentence_transformers import SentenceTransformer

# 임베딩 모델 선택
model_options = {
    "all-MiniLM-L6-v2": {
        "size": 384,
        "speed": "빠름",
        "quality": "좋음",
        "use": "대부분의 RAG"
    },
    "all-mpnet-base-v2": {
        "size": 768,
        "speed": "중간",
        "quality": "매우 좋음",
        "use": "고정확도 필요"
    },
    "text-embedding-3-small": {
        "size": 1536,
        "speed": "중간",
        "quality": "최고",
        "use": "최고 품질 필요 (유료)"
    }
}

# 로드
model = SentenceTransformer('all-MiniLM-L6-v2')

# Embedding 생성
embeddings = model.encode([chunk["text"] for chunk in chunks])
# embeddings.shape: (num_chunks, 384)
```

### Batch 처리

```python
def embed_chunks(chunks, model, batch_size=32):
    """대량의 청크를 효율적으로 임베딩"""
    all_embeddings = []
    
    for i in range(0, len(chunks), batch_size):
        batch = chunks[i:i+batch_size]
        texts = [chunk["text"] for chunk in batch]
        
        embeddings = model.encode(texts, batch_size=batch_size)
        all_embeddings.extend(embeddings)
        
        print(f"Processed {i+len(batch)}/{len(chunks)}")
    
    return all_embeddings
```

### Embedding 저장

```python
import numpy as np
from pinecone import Pinecone

# Pinecone 초기화
pc = Pinecone(api_key="xxx")
index = pc.Index("documents")

# 벡터 저장
vectors_to_upsert = []
for i, chunk in enumerate(chunks):
    vectors_to_upsert.append({
        "id": chunk["id"],
        "values": embeddings[i],
        "metadata": chunk["metadata"]
    })

# 배치 저장
index.upsert(vectors=vectors_to_upsert)
```

---

## Retrieval

### 검색 구현

```python
def retrieve_documents(query, model, index, top_k=5):
    """질문과 유사한 문서 검색"""
    
    # 1. 질문 임베딩
    query_embedding = model.encode(query)
    
    # 2. 벡터 검색
    results = index.query(
        vector=query_embedding,
        top_k=top_k,
        include_metadata=True
    )
    
    # 3. 결과 정렬
    retrieved_docs = []
    for match in results["matches"]:
        retrieved_docs.append({
            "id": match["id"],
            "score": match["score"],
            "metadata": match["metadata"],
            "source": match["metadata"].get("source")
        })
    
    return retrieved_docs
```

### "검색 결과를 prompt에 얼마나 넣을 것인가?"

**검색 결과의 개수와 길이는 LLM의 컨텍스트 윈도우와 질문 복잡도에 따라 조정**한다.

```text
문서 개수:
- 단순 질문: 3-5개
- 복잡한 질문: 5-10개
- 종합적 답변: 10-20개

각 문서 길이:
- 토큰 제한 고려
- 총 토큰 = 질문 + 문서 + 여유(1000-2000)
- 예: 100K 컨텍스트 window
      질문: 100 토큰
      여유: 2000 토큰
      문서: 98K 토큰 (약 300단어 × 30개)
```

### 검색 결과 순서 조정

```python
def rerank_results(query, retrieved_docs, model):
    """검색 결과 재순위 매김"""
    
    # Cross-encoder를 사용해 재순위
    from sentence_transformers import CrossEncoder
    
    reranker = CrossEncoder('cross-encoder/qnli-distilroberta-base')
    
    # 쌍을 만들어 유사도 계산
    pairs = [(query, doc["text"]) for doc in retrieved_docs]
    scores = reranker.predict(pairs)
    
    # 점수로 재정렬
    for i, doc in enumerate(retrieved_docs):
        doc["rerank_score"] = scores[i]
    
    retrieved_docs.sort(key=lambda x: x["rerank_score"], reverse=True)
    return retrieved_docs
```

---

## Answer Generation

### LLM 프롬프트 구성

```python
def build_rag_prompt(query, retrieved_docs, max_docs=5):
    """검색 결과를 포함한 RAG 프롬프트 구성"""
    
    # 상위 문서만 선택
    docs = retrieved_docs[:max_docs]
    
    # 문서 컨텍스트 구성
    context = ""
    for i, doc in enumerate(docs, 1):
        context += f"\n[문서 {i} - {doc['metadata']['source']}]\n"
        context += f"{doc['text']}\n"
    
    # 프롬프트 구성
    prompt = f"""다음은 질문에 답변하기 위한 관련 문서 발췌다:

{context}

---

사용자 질문: {query}

위 문서를 기반으로 정확하게 답변해. 
만약 문서에 답변이 없으면 "문서에 명시되지 않음"이라고 말해.
답변할 때 어떤 문서를 참고했는지 표시해."""
    
    return prompt, docs
```

### LLM으로 답변 생성

```python
from anthropic import Anthropic

def generate_answer(query, retrieved_docs):
    """RAG를 통해 최종 답변 생성"""
    
    client = Anthropic()
    
    # 프롬프트 구성
    prompt, docs = build_rag_prompt(query, retrieved_docs)
    
    # LLM 호출
    response = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[
            {"role": "user", "content": prompt}
        ]
    )
    
    answer = response.content[0].text
    
    return {
        "answer": answer,
        "sources": [d["metadata"]["source"] for d in docs],
        "doc_count": len(docs)
    }
```

### "답변에 근거 출처를 어떻게 표시할 것인가?"

**프롬프트에서 출처 표시를 강제하고, 답변 후처리로 출처 추출**한다.

```python
def add_citations(answer_text, retrieved_docs):
    """답변에 출처 추가"""
    
    # 간단한 방식: 문서별 번호 추가
    citations = []
    for i, doc in enumerate(retrieved_docs[:5], 1):
        citations.append({
            "num": i,
            "source": doc["metadata"]["source"],
            "url": f"file://{doc['metadata']['source']}"
        })
    
    # 형식:
    # "답변 내용 [1]"
    # [1] source.md
    
    formatted_answer = answer_text
    citation_section = "\n\n참고 문서:\n"
    for cite in citations:
        citation_section += f"[{cite['num']}] {cite['source']}\n"
    
    return formatted_answer + citation_section
```

---

## 구현 예시

### 전체 파이프라인

```python
from sentence_transformers import SentenceTransformer
from anthropic import Anthropic
from pathlib import Path

class RAGSystem:
    def __init__(self, model_name="all-MiniLM-L6-v2"):
        self.embedding_model = SentenceTransformer(model_name)
        self.llm_client = Anthropic()
        self.chunks = []
        self.embeddings = []
        self.index = None
    
    def load_documents(self, doc_dir):
        """문서 로드"""
        documents = []
        for md_file in Path(doc_dir).glob("*.md"):
            with open(md_file, "r") as f:
                documents.append({
                    "source": md_file.name,
                    "content": f.read()
                })
        return documents
    
    def chunk_documents(self, documents, chunk_size=300, overlap=50):
        """청킹"""
        from langchain.text_splitter import RecursiveCharacterTextSplitter
        
        splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=overlap
        )
        
        self.chunks = []
        for doc in documents:
            splits = splitter.split_text(doc["content"])
            for i, text in enumerate(splits):
                self.chunks.append({
                    "id": f"{doc['source']}_{i}",
                    "text": text,
                    "source": doc["source"]
                })
    
    def create_embeddings(self):
        """임베딩 생성"""
        texts = [chunk["text"] for chunk in self.chunks]
        self.embeddings = self.embedding_model.encode(texts)
    
    def query(self, question, top_k=5):
        """질문에 답변"""
        
        # 1. 검색
        query_embedding = self.embedding_model.encode(question)
        
        # 2. 코사인 유사도 계산
        import numpy as np
        scores = np.dot(self.embeddings, query_embedding)
        top_indices = np.argsort(-scores)[:top_k]
        
        retrieved = [self.chunks[i] for i in top_indices]
        
        # 3. 답변 생성
        prompt = self._build_prompt(question, retrieved)
        
        response = self.llm_client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=1024,
            messages=[{"role": "user", "content": prompt}]
        )
        
        return response.content[0].text

# 사용
rag = RAGSystem()
docs = rag.load_documents("docs")
rag.chunk_documents(docs)
rag.create_embeddings()

answer = rag.query("NAT는 무엇인가?")
print(answer)
```

---

## 평가 및 최적화

### 성능 평가

```python
def evaluate_rag(test_questions, ground_truths):
    """RAG 시스템 평가"""
    
    from rouge_score import rouge_scorer
    
    results = {
        "retrieval": [],
        "generation": []
    }
    
    for question, true_answer in zip(test_questions, ground_truths):
        # 검색 평가
        retrieved = rag.retrieve(question)
        retrieval_recall = calculate_recall(retrieved, ground_truths[question])
        results["retrieval"].append(retrieval_recall)
        
        # 생성 평가
        generated = rag.query(question)
        scorer = rouge_scorer.RougeScorer(['rouge1', 'rougeL'])
        scores = scorer.score(true_answer, generated)
        results["generation"].append(scores['rougeL'].fmeasure)
    
    print(f"평균 Retrieval Recall: {np.mean(results['retrieval']):.2%}")
    print(f"평균 ROUGE-L: {np.mean(results['generation']):.2%}")
```

### 최적화 전략

```text
1. 청크 크기 튜닝
   - 측정: retrieval accuracy
   - 시작: 300-500단어
   - 조정: 성능에 따라 ±100단어

2. 임베딩 모델 선택
   - 트레이드오프: 정확도 vs 속도
   - 평가: Retrieval precision@k

3. 프롬프트 최적화
   - 문서 개수: 3-10개 조정
   - 지시사항: 명확함 강화
   - Few-shot 예시 추가

4. 검색 결과 재순위
   - Cross-encoder 사용
   - BM25 + 벡터 검색 조합
   - 필터링 규칙 추가
```

---

## 정리

| 단계 | 설명 | 핵심 요소 |
|------|------|----------|
| 문서 수집 | 다양한 소스에서 문서 수집 | 포맷 다양성 |
| Chunking | 문서를 작은 단위로 분할 | 크기, 겹침 |
| Embedding | 텍스트를 벡터로 변환 | 모델 선택 |
| Retrieval | 질문과 유사한 문서 검색 | 상위 K개 |
| Generation | 검색 결과 기반 답변 생성 | 프롬프트 구성 |

---

## 다음 단계

### 개선 아이디어

```text
1. 하이브리드 검색 (키워드 + 벡터)
2. 멀티 턴 대화 (대화 이력 유지)
3. 실시간 문서 업데이트
4. 사용자 피드백 수집
5. 생성 검증 (Fact-checking)
```
