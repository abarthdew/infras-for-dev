# Data Pipeline

## 목차
- [Data Pipeline의 역할](#data-pipeline의-역할)
- [Source](#source)
- [Ingestion](#ingestion)
- [Transformation](#transformation)
- [Storage](#storage)
- [Orchestration](#orchestration)
- [Data Quality](#data-quality)
- [Monitoring](#monitoring)

---

## Data Pipeline의 역할

**Data Pipeline은 다양한 원천에서 원시 데이터를 수집하고, 정제하고, 변환해서 분석이나 서비스에 쓸 수 있는 형태로 만드는 프로세스**다.

```text
Source (원본 데이터)
    ↓
Ingestion (수집)
    ↓
Transformation (정제, 변환)
    ↓
Storage (저장)
    ↓
Serving (분석, BI, 모델)
```

### 파이프라인의 필요성

```text
원시 데이터 문제:
- 다양한 포맷 (CSV, JSON, Parquet, Binary)
- 다양한 위치 (DB, API, 파일, 로그)
- 오류와 중복 (정제 필요)
- 느린 접근 속도
- 불완전한 정보

파이프라인 후:
- 통일된 포맷
- 중앙 저장소 (Data Lake, Warehouse)
- 검증된 데이터
- 빠른 쿼리
- 완전한 정보
```

---

## Source

### Source 종류

```text
Database (OLTP)
├─ MySQL, PostgreSQL
├─ MongoDB, DynamoDB
└─ 현재 상태 데이터

File System
├─ CSV, JSON, Parquet
├─ 로그 파일
└─ 스냅샷 데이터

API
├─ REST API
├─ GraphQL
└─ 실시간 데이터 스트림

Event Stream
├─ Kafka, Kinesis
├─ 실시간 이벤트
└─ 높은 처리량
```

### CDC (Change Data Capture)

```text
문제 (전체 스캔):
SELECT * FROM users WHERE updated_at > '2026-05-23'
→ 테이블이 크면 느림
→ 필요한 데이터만 부분 복제 어려움

해결 (CDC):
MySQL Binlog → Debezium → Kafka
→ 변경된 행만 스트리밍
→ 실시간, 효율적

흐름:
1. DB에서 INSERT, UPDATE, DELETE 발생
2. CDC 도구가 감지
3. 변경사항을 이벤트로 변환
4. Kafka 등으로 전송
5. 다운스트림 시스템이 수신
```

---

## Ingestion

### "ETL과 ELT는 무엇이 다른가?"

**ETL (Extract-Transform-Load)은 수집 후 변환하고 저장, ELT (Extract-Load-Transform)은 수집 후 바로 저장하고 나중에 변환**한다.

```text
ETL (Traditional):
Source
    ↓ Extract
Raw Data
    ↓ Transform (별도 서버에서)
- 정제, 검증
- 조인, 집계
- 비즈니스 로직 적용
Transformed Data
    ↓ Load
Data Warehouse
    ↓
분석/BI

ELT (Modern):
Source
    ↓ Extract
Raw Data
    ↓ Load
Data Lake (빠르게, 그대로)
    ↓ Transform (Warehouse 내에서 SQL/Spark로)
- 정제, 검증
- 조인, 집계
- 비즈니스 로직 적용
Data Warehouse
    ↓
분석/BI
```

### ETL vs ELT 비교

```text
ETL:
장점: 저장된 데이터는 이미 정제됨, 저장 공간 효율
단점: 느린 로딩 (변환 시간), 스키마 변경 어려움

ELT:
장점: 빠른 로딩, 유연성 높음 (원본 보존), 스키마 진화 쉬움
단점: 원본 데이터 많음 (저장 비용), 사용자가 변환 관리

추세: ELT가 더 인기 (클라우드 데이터웨어 비용 감소로)
```

### Ingestion 도구

```text
일괄 처리 (Batch):
- Talend, Informatica
- 시간대 기반 배치
- 낮은 지연

실시간 (Real-time):
- Kafka, Kinesis
- 즉시 수신
- 높은 처리량

하이브리드:
- Fivetran, Stitch
- 관리형 수집
- 자동 스키마 진화
```

---

## Transformation

### 변환 작업

```text
정제 (Cleaning):
- NULL 값 처리
- 중복 제거
- 이상치 탐지

정규화 (Normalization):
- 텍스트 형식 통일
- 날짜 포맷 통일
- 단위 변환 (USD → KRW)

강화 (Enrichment):
- 외부 데이터 조인
- 참조 데이터 추가
- 계산 필드 생성

집계 (Aggregation):
- 시간별 그룹화
- 사용자별 요약
- 통계 계산
```

### "schema evolution은 왜 어렵나?"

**스키마가 변경되면 기존 데이터와의 호환성, 파이프라인 재처리 등 여러 문제가 발생**한다.

```text
예시: users 테이블에 address 컬럼 추가

문제 1: 역호환성
- 기존 파일/스냅샷: address 없음
- 새 코드: address 기대
- Null 처리 필요

문제 2: 재처리
- 과거 데이터 address = NULL
- 재처리 해야 하나? (비용)

문제 3: 다운스트림
- Dashboard: address 컬럼 참조
- 대시보드 깨짐?
- 자동으로 NULL 처리?

해결책:
- Union type (Old | New schema)
- 점진적 롤아웃
- 버전 관리 (v1, v2)
- 마이그레이션 스크립트
```

### 변환 기술

```text
SQL:
- 간단, 빠름
- dbt (데이터 빌드 도구)

Python/Scala:
- 복잡한 로직
- PySpark, Scala

통합 도구:
- dbt, Talend
- UI 기반 설정
```

---

## Storage

### 저장 계층

```text
Data Lake (원본 저장):
- 모든 형식의 데이터
- S3, HDFS, ADLS
- 저렴한 저장 비용
- 구조화되지 않음

Data Warehouse (분석용):
- 정제되고 구조화된 데이터
- Snowflake, BigQuery, Redshift
- 비용 높음
- 빠른 쿼리
```

### 데이터 포맷

```text
CSV (간단):
- 텍스트
- 호환성 좋음
- 크기 큼, 느림

Parquet (추천):
- 컬럼 기반
- 압축 좋음
- 빠른 쿼리
- Spark, 데이터 분석 친화

ORC:
- 컬럼 기반
- 매우 효율적
- Hive 최적화

JSON (반정형):
- 유연
- 중첩 구조 지원
- 크기 큼
```

---

## Orchestration

### Orchestration이란?

**파이프라인의 각 단계를 예약하고, 순서대로 실행하고, 실패를 처리하는 스케줄 관리**다.

```text
Day 1 09:00 - Ingest users from MySQL
    ↓ (완료 후)
Day 1 10:00 - Transform (join, clean)
    ↓ (완료 후)
Day 1 11:00 - Load to Warehouse
    ↓ (완료 후)
Day 1 12:00 - Generate report
    ↓ (완료 후)
Day 1 13:00 - Alert if failed

실패:
- Step 3이 실패
- 자동 재시도 또는 알림
- 이후 Step 4, 5 자동 취소
```

### Orchestration 도구

```text
Apache Airflow:
- 오픈소스 (Python)
- DAG (Directed Acyclic Graph) 기반
- 운영 복잡

Prefect, Dagster:
- 모던한 대안
- 더 간단한 문법

Cloud 관리형:
- Google Cloud Composer (Airflow 기반)
- AWS Glue, Step Functions
- Azure Data Factory
```

### DAG 예시 (Airflow)

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def extract():
    # MySQL에서 데이터 추출
    pass

def transform():
    # 데이터 정제 및 변환
    pass

def load():
    # Warehouse에 로드
    pass

dag = DAG('daily_pipeline', start_date=datetime(2026, 1, 1), schedule='@daily')

t1 = PythonOperator(task_id='extract', python_callable=extract, dag=dag)
t2 = PythonOperator(task_id='transform', python_callable=transform, dag=dag)
t3 = PythonOperator(task_id='load', python_callable=load, dag=dag)

t1 >> t2 >> t3  # 순차 실행
```

---

## Data Quality

### "데이터 품질은 어디에서 검증해야 하는가?"

**데이터 품질은 각 단계에서 검증해야 한다. 초기에 잘못된 데이터가 들어오면 모든 다운스트림에 영향**을 미친다.**

```text
Source 검증 (Ingestion 전):
- 파일 포맷 확인
- 필수 필드 존재 확인
- 날짜 형식 유효성

Ingestion 검증 (로드 후):
- Record count 확인
- 기본키 중복 확인
- NULL 비율 확인

Transformation 검증 (변환 후):
- 비즈니스 규칙 확인 (예: 나이 > 0)
- 조인 성공률 확인
- 이상치 탐지

Serving 검증 (사용 전):
- Dashboard 쿼리 성능
- 데이터 일관성
- 사용자 피드백

원칙:
- 빠를수록 좋음 (조기 발견)
- 각 단계마다 (누적 방지)
- 자동화 (수동 확인 불가)
```

### Data Quality 도구

```text
Great Expectations:
- 데이터 테스트 프레임워크
- 기대값 정의 후 검증

Monte Carlo Data:
- 데이터 관찰성
- 이상치 자동 감지

Soda SQL:
- SQL 기반 검증
```

---

## Monitoring

### "pipeline failure는 어떻게 감지하고 복구할 것인가?"

**파이프라인 실패는 자동으로 감지하고, 재시도 또는 알림으로 대응**해야 한다.

```text
모니터링 항목:

실행 시간:
- 예상 10분, 실제 30분? → 느린 것으로 감지
- 필터링 조건 변경? 데이터 증가?

데이터 품질:
- 예상 100K 행, 실제 50K? → 데이터 손실
- 새로운 NULL 값? → 스키마 변경

완료 여부:
- 파이프라인이 끝났나?
- 중간에 멈춤?
```

### 실패 처리 전략

```text
자동 재시도:
- 일시적 오류 (네트워크 타임아웃) → 자동 재시도
- 영구적 오류 (스키마 불일치) → 알림

데드레터 큐:
- 실패한 메시지를 별도 큐에 저장
- 나중에 수동 검토
- 재처리

Backfill:
- 과거 데이터 다시 처리
- 스키마 변경 후 과거 데이터 재변환

Circuit Breaker:
- 계속 실패하면 멈춤 (비용 절감)
- 일정 시간 후 재시도
```

### Alerting

```text
Critical (즉시 대응):
- 파이프라인 실패 (3회 이상 재시도)
- 데이터 손실 (행 수 90% 이상 감소)

Warning (주의):
- 느린 실행 (예상의 2배)
- NULL 값 급증

Info (참고):
- 성공 완료
- 데이터 추가됨
```

---

## 정리

| 단계 | 설명 |
|------|------|
| Source | 원본 데이터 위치 |
| Ingestion | 데이터 수집 (Batch, Real-time) |
| Transformation | 정제, 변환, 강화 |
| Storage | Data Lake 또는 Warehouse |
| Orchestration | 스케줄 및 순서 관리 |
| Data Quality | 검증 및 모니터링 |

---

## Related Notes

- [Batch Processing](../batch-processing/overview.md)
- [Streaming](../streaming/overview.md)
