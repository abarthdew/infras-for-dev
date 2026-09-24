# Batch Processing

## 목차
- [Batch Processing의 역할](#batch-processing의-역할)
- [Batch Job](#batch-job)
- [Scheduler](#scheduler)
- [Input/Output](#inputoutput)
- [Retry](#retry)
- [Idempotency](#idempotency)
- [Backfill](#backfill)

---

## Batch Processing의 역할

**Batch Processing은 데이터를 실시간이 아니라 일정 단위로 모아 한 번에 처리하는 방식**이다. 일별 정산, 리포트 생성, 대량 ETL에 효율적이다.

```text
"batch와 streaming은 무엇이 다른가?"

Batch Processing:
- 시간 또는 용량 기준으로 모아서 처리
- 예: 매일 02:00에 전날 데이터 처리
- 처리량 많음, 지연 있음 (수 시간)

Real-time Streaming:
- 데이터가 오는 즉시 처리
- 예: 실시간 이벤트 분석
- 처리량 적음, 지연 없음 (밀리초)
```

### Batch vs Streaming 비교

```text
Batch:
├─ 처리량: 매우 높음 (수십억 레코드)
├─ 지연: 수 시간 (OK)
├─ 비용: 저렴 (대량 처리 효율)
├─ 구현: 간단
└─ 예: 일일 정산, 리포트, 대용량 ETL

Streaming:
├─ 처리량: 중간 (수백만 레코드/초)
├─ 지연: 밀리초 (필수)
├─ 비용: 비쌈 (24/7 실행)
├─ 구현: 복잡
└─ 예: 실시간 알림, 대시보드, 추천
```

---

## Batch Job

### Batch Job이란?

```text
정의:
- 정해진 시간에 시작
- 입력 데이터 처리
- 결과 저장
- 완료 또는 실패
```

### 구현 예시

```python
# PySpark batch job
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("daily_batch").getOrCreate()

# 입력: 전날 주문 데이터
orders = spark.read.parquet("s3://data-lake/orders/dt=2026-05-22/")

# 처리
result = orders.groupBy("user_id") \
    .agg({"amount": "sum", "order_count": "count"}) \
    .withColumn("dt", "2026-05-22")

# 출력: 결과 저장
result.write.parquet("s3://warehouse/daily_sales/dt=2026-05-22/")

# 로그
print(f"Processed {orders.count()} orders")
```

---

## Scheduler

### 스케줄 관리

```text
Cron 방식:
0 2 * * * python /opt/batch/daily_job.py
↑ ↑ ↑ ↑ ↑
분 시 일 월 요일

매일 02:00에 실행

Orchestration (Airflow):
DAG로 정의
- 선행 작업 완료 후 실행
- 재시도 정책 지정
- 모니터링
```

### 실행 패턴

```text
일일 배치:
02:00 - 전날 데이터 처리

시간별 배치:
매 정시 - 지난 시간 데이터 처리

주간 배치:
일요일 03:00 - 일주일 데이터 처리

임시 배치:
필요할 때 수동 실행 (backfill)
```

---

## Input/Output

### 입력

```text
Batch 처리 범위 정의:
- 날짜 범위: 2026-05-22
- 데이터 위치: s3://data-lake/orders/dt=2026-05-22/
- 파티션: 날짜별, 지역별 분할

Partition:
- 각 배치는 명확한 데이터 범위
- 재처리 시 해당 파티션만 처리
- 병렬 처리 가능
```

### 출력

```text
결과 저장:
1. 데이터 저장 (Parquet)
   - s3://warehouse/daily_sales/dt=2026-05-22/
   
2. 메타데이터 기록
   - 처리 완료 시간
   - 처리된 레코드 수
   - 체크섬
   
3. 상태 업데이트
   - success / failure
   - 알림 전송
```

---

## Retry

### "실패한 batch job은 어떻게 재시도해야 하는가?"

**배치 재시도는 일시적 오류와 영구적 오류를 구분하고, 지수 백오프를 사용해야 한다.**

```text
재시도 전략:

일시적 오류 (재시도 가능):
- 네트워크 타임아웃 → 자동 재시도
- 일시 리소스 부족 → 대기 후 재시도
- 외부 API 일시 오류

영구적 오류 (재시도 불가):
- 스키마 불일치 → 코드 수정 필요
- 파일 없음 → 입력 데이터 확인
- 권한 부족 → 설정 수정

재시도 정책:
1차: 1분 후
2차: 3분 후 (지수 백오프)
3차: 9분 후
4차: 27분 후
포기: 4회 이상 실패
```

### 구현

```python
import time
from functools import wraps

def retry_on_failure(max_attempts=3, backoff_factor=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts:
                        raise
                    wait_time = (backoff_factor ** (attempt - 1)) * 60
                    print(f"Attempt {attempt} failed, retrying in {wait_time}s")
                    time.sleep(wait_time)
        return wrapper
    return decorator

@retry_on_failure(max_attempts=4, backoff_factor=3)
def batch_job():
    # 배치 작업
    pass
```

---

## Idempotency

### "idempotent job은 왜 중요한가?"

**Idempotent 배치는 같은 입력으로 여러 번 실행해도 같은 결과를 내므로, 재시도 시에도 안전**하다.**

```text
비 Idempotent (위험):
매번 실행할 때마다 결과가 다름

배치 1차: INSERT INTO sales SELECT ... FROM orders WHERE dt='2026-05-22'
결과: 1000 행 추가

배치 2차 (재시도): INSERT INTO sales ... (같은 쿼리)
결과: 1000 행 추가 (총 2000행, 중복!)
→ 데이터 중복!

Idempotent (안전):
같은 입력 → 항상 같은 결과

방법 1: DELETE 후 INSERT
DELETE FROM sales WHERE dt='2026-05-22'
INSERT INTO sales ... WHERE dt='2026-05-22'
→ 몇 번 실행해도 같은 결과

방법 2: UPSERT (INSERT OR UPDATE)
INSERT INTO sales ... WHERE dt='2026-05-22'
ON CONFLICT UPDATE ...
→ 중복 제거
```

### 구현

```python
# Parquet 기반 (파티션 덮어쓰기)
result.write \
    .mode("overwrite") \
    .parquet(f"s3://warehouse/sales/dt={date}/")

# 파티션을 완전히 덮어씀
# 재시도: DELETE + INSERT 동일
```

---

## Backfill

### "backfill은 언제 필요한가?"

**Backfill은 과거 데이터를 다시 처리해야 할 때 사용한다. 코드 변경, 데이터 복구, 누락 처리 등.**

```text
예시 1: 버그 수정
- 코드 버그로 2026-05-15~20 데이터가 잘못됨
- 코드 수정 후
- Backfill: 2026-05-15~20 데이터 재처리

예시 2: 새로운 필드 추가
- user_grade 필드 추가 필요
- 과거 데이터에도 grade 계산
- Backfill: 전체 기간 다시 처리

예시 3: 배치 누락
- 2026-05-18 배치가 실패했음
- Backfill: 2026-05-18만 재처리

실행:
./batch_job.py --date=2026-05-15 --end-date=2026-05-20
```

### Backfill 안전성

```text
일반 배치:
- 미리 정해진 시간에 자동 실행
- 입력 데이터가 완전

Backfill:
- 수동으로 특정 기간 지정
- 입력 데이터가 여전히 있나?
- 이미 처리된 기간 다시 처리 (중복?)

안전성 확보:
✓ Idempotent 구현 (필수)
✓ 테스트 환경에서 먼저 실행
✓ 결과 검증 (행 수, 체크섬)
✓ 느린 실행으로 리소스 모니터링
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Batch Job | 정해진 시간에 데이터 일괄 처리 |
| Scheduler | 배치 실행 시간 및 빈도 관리 |
| Retry | 실패 시 자동 재시도 |
| Idempotency | 여러 번 실행해도 같은 결과 |
| Backfill | 과거 데이터 재처리 |

---

## Related Notes

- [Data Pipeline](../data-pipeline/overview.md)
- [Streaming](../streaming/overview.md)
