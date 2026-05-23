# Batch Processing

## 목차
- batch job
- scheduler
- input/output
- retry
- idempotency
- backfill

## 기초 개념
배치 처리는 데이터를 실시간이 아니라 일정 단위로 모아 처리하는 방식이다. 일별 정산, 리포트 생성, 대량 ETL에 자주 쓰인다.

```text
source data -> batch job -> transformed data
```

## 간단한 예시
```text
매일 02:00
-> 전날 주문 데이터 집계
-> 매출 리포트 테이블 생성
```

## 반드시 알아야 할 질문
- batch와 streaming은 무엇이 다른가?
- 실패한 batch job은 어떻게 재시도해야 하는가?
- idempotent job은 왜 중요한가?
- backfill은 언제 필요한가?
