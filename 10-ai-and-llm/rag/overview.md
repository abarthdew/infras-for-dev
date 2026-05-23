# RAG

## 목차
- retrieval
- embedding
- vector search
- context injection
- grounding
- evaluation

## 기초 개념
RAG는 LLM이 답변하기 전에 외부 문서를 검색해 관련 내용을 prompt에 넣는 방식이다. 모델의 내부 기억에만 의존하지 않고 근거 문서를 활용한다.

```text
question -> retrieve documents -> prompt with context -> answer
```

## 간단한 예시
```text
사용자 질문
-> 사내 문서 검색
-> 관련 문단 5개를 prompt에 포함
-> 답변 생성
```

## 반드시 알아야 할 질문
- RAG는 fine-tuning과 무엇이 다른가?
- chunk size는 검색 품질에 어떤 영향을 주는가?
- embedding model은 왜 중요하다?
- retrieved context가 틀리면 답변은 어떻게 망가지는가?
