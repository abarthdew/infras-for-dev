# Vector Database

## 목차
- embedding vector
- similarity search
- index
- metadata filtering
- recall과 latency

## 기초 개념
벡터 데이터베이스는 텍스트나 이미지 같은 데이터를 embedding vector로 저장하고, 의미적으로 가까운 항목을 찾는 데 특화된 저장소다.

```text
document -> embedding -> vector index -> similarity search
```

## 간단한 예시
```text
"리눅스 포트란?"
-> query embedding
-> 비슷한 문서 chunk 검색
```

## 반드시 알아야 할 질문
- embedding은 무엇을 숫자로 표현하는가?
- cosine similarity는 무엇을 비교하는가?
- ANN index는 왜 정확도와 속도 trade-off가 있는가?
- metadata filter는 언제 필요하다?
