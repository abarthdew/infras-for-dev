# Elasticsearch

## 목차
- 검색 엔진의 역할
- index와 document
- inverted index
- analyzer
- full-text search
- aggregation

## 기초 개념
Elasticsearch는 검색과 분석에 특화된 분산 검색 엔진이다. 텍스트를 분석해 inverted index를 만들고, 빠른 전문 검색과 집계를 제공한다.

```text
document -> analyze -> inverted index -> search
```

## 간단한 예시
```json
{
  "title": "linux network basics",
  "tags": ["linux", "network"]
}
```

## 반드시 알아야 할 질문
- 관계형 DB 검색과 전문 검색은 무엇이 다른가?
- inverted index는 왜 검색을 빠르게 하는가?
- analyzer는 왜 언어별로 달라질 수 있는가?
- Elasticsearch는 원본 DB를 대체하는가?
