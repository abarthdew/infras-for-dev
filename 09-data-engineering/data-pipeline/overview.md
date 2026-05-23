# Data Pipeline

## 목차
- source
- ingestion
- transformation
- storage
- orchestration
- data quality

## 기초 개념
데이터 파이프라인은 원천 데이터가 분석이나 서비스에 쓰일 수 있는 형태로 이동하고 변환되는 흐름이다.

```text
source -> ingest -> transform -> store -> serve
```

## 간단한 예시
```text
application DB
-> CDC
-> data lake
-> batch transform
-> warehouse table
-> dashboard
```

## 반드시 알아야 할 질문
- ETL과 ELT는 무엇이 다른가?
- 데이터 품질은 어디에서 검증해야 하는가?
- schema evolution은 왜 어렵나?
- pipeline failure는 어떻게 감지하고 복구할 것인가?
