# Project: RAG Demo

## 목차
- 목표
- 문서 수집
- chunking
- embedding
- retrieval
- answer generation

## 기초 개념
이 프로젝트는 문서를 작은 chunk로 나누고 embedding해 검색한 뒤, 검색 결과를 LLM prompt에 넣어 답변하는 RAG 흐름을 실습한다.

```text
documents -> chunks -> embeddings -> vector search -> LLM answer
```

## 간단한 예시
```text
질문: "NAT는 무엇인가?"
-> 관련 문서 chunk 검색
-> chunk를 근거로 답변 생성
```

## 반드시 알아야 할 질문
- chunk 크기는 검색 품질에 어떤 영향을 주는가?
- 검색 결과를 prompt에 얼마나 넣을 것인가?
- 답변에 근거 출처를 어떻게 표시할 것인가?
