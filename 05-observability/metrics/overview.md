# Metrics

## 목차
- [Metrics의 역할](#metrics의-역할)
- [Metric Type](#metric-type)
- [RED Method](#red-method)
- [USE Method](#use-method)
- [SLI와 SLO](#sli와-slo)
- [Cardinality](#cardinality)
- [Metric 설계](#metric-설계)

---

## Metrics의 역할

**Metrics는 시스템 상태를 숫자로 표현한 시계열 데이터**다. 로그는 개별 사건의 상세 정보, 메트릭은 시스템 전체의 집계 통계를 제공한다.

```text
Logs (개별 사건):
- 특정 요청이 언제 실패했는가?
- 어떤 에러가 발생했는가?

Metrics (집계 통계):
- 시간대별 에러율은?
- 평균 응답 시간은?
- CPU 사용량은?
- 초당 요청 수는?

Traces (요청 흐름):
- 이 요청이 어느 서비스를 거쳤는가?
- 각 단계에서 시간은 얼마나 걸렸는가?
```

### Metrics의 필요성

```text
실시간 모니터링:
- 접속자 100명 vs 10만 명 → 즉시 파악
- CPU 50% vs 95% → 임계값으로 알림

성능 분석:
- 이번 주 vs 지난 주 비교
- 배포 전후 성능 변화

용량 계획:
- 일일 최대 요청 수: 1M
- 필요 서버 수: ?
- 진행 추세로 언제까지 충분? (capacity planning)

비용 최적화:
- 메모리 사용: 50% → 자동 스케일 다운 가능
- 비용 절감 기회 발굴
```

---

## Metric Type

### Metric의 종류

```text
Counter (계수기)
- 계속 증가하는 값 (절대 감소하지 않음)
- 예: 총 요청 수, 총 에러 수, 총 바이트 처리
- 연산: Rate = delta / time (초당 요청 수)

Gauge (게이지)
- 현재 값 (증감 가능)
- 예: 현재 메모리 사용, 활성 연결 수, CPU 사용률
- 특징: 순간값 (스냅샷)

Histogram (히스토그램)
- 값의 분포 (버킷으로 나눔)
- 예: 응답 시간 분포 (0-10ms, 10-50ms, 50-100ms...)
- 연산: 퍼센타일 (p50, p95, p99) 계산 가능

Summary (요약)
- 값의 통계 요약 (평균, 합, 개수)
- 예: 응답 시간의 평균, 합계, 개수
```

### 예시

```
Counter:
http_requests_total{method="GET", status="200"} 12345
→ GET 요청이 12345건 성공

Gauge:
memory_usage_bytes{instance="server1"} 2147483648
→ server1의 메모리 사용: 2GB

Histogram (내부적으로 여러 버킷):
http_request_duration_seconds_bucket{le="0.1"} 1000
http_request_duration_seconds_bucket{le="0.5"} 5000
http_request_duration_seconds_bucket{le="1.0"} 8000
→ 100ms 이하: 1000건, 500ms 이하: 5000건...
→ p50 ≈ 100ms, p95 ≈ 500ms 계산 가능
```

---

## RED Method

### "request rate, error rate, duration은 왜 기본 지표인가?"

**RED Method (Rate, Errors, Duration)는 서비스의 건강성을 빠르게 판단하는 세 가지 핵심 지표**다.

```text
Request Rate (처리량)
- 초당 얼마나 많은 요청을 처리하는가?
- 정상 범위: 1000 RPS
- 급감 → 문제 가능성
- 급증 → 부하 증가

Error Rate (에러율)
- 요청 중 몇 %가 실패하는가?
- 정상: 0.1% 이하
- 1% → 문제 (100개 중 1개 실패)
- 5% → 심각 (20개 중 1개 실패)

Duration (응답 시간)
- 평균 얼마나 걸리는가?
- 정상: 50-100ms
- 500ms → 느림
- 5초 → 타임아웃 가능
```

### RED 메트릭 수집

```python
from prometheus_client import Counter, Histogram
import time

# Request Rate
request_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Duration
request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

# 미들웨어
def middleware(app):
    def handle_request(environ, start_response):
        start_time = time.time()
        method = environ['REQUEST_METHOD']
        path = environ['PATH_INFO']
        
        try:
            # 요청 처리
            response = app(environ, start_response)
            status = 200
        except Exception as e:
            status = 500
            response = str(e)
        
        # 메트릭 기록
        duration = time.time() - start_time
        request_total.labels(method=method, endpoint=path, status=status).inc()
        request_duration.labels(method=method, endpoint=path).observe(duration)
        
        return response
    
    return handle_request
```

### RED 대시보드

```
대시보드 항목:

[Rate]
- Total RPS
- RPS by endpoint
- RPS trend (1h, 24h)

[Errors]
- Error rate (%)
- Errors by status code (4xx, 5xx)
- Error trend

[Duration]
- p50 latency
- p95 latency
- p99 latency
- Duration by endpoint
```

---

## USE Method

### USE Method란?

**USE Method (Utilization, Saturation, Errors)는 인프라/자원의 건강성을 판단하는 세 가지 지표**다.

```text
Utilization (사용률)
- CPU 사용률
- 메모리 사용률
- 디스크 사용률
- 네트워크 대역폭 사용률

Saturation (포화도)
- 처리 대기열 길이
- CPU 대기 시간
- I/O 대기 시간

Errors (에러)
- 디스크 읽기 에러
- 네트워크 패킷 손실
- 메모리 할당 실패
```

### USE vs RED

```text
서비스 모니터링 → RED
┌─ Request Rate
├─ Error Rate
└─ Duration

인프라 모니터링 → USE
┌─ Utilization (CPU, Memory, Disk)
├─ Saturation (Queue, Wait time)
└─ Errors (System errors)
```

### USE 메트릭 수집

```bash
# CPU
cpu_usage_percent: 45%

# Memory
memory_used_bytes: 4GB / 8GB = 50%

# Disk
disk_used_bytes: 450GB / 500GB = 90%  # ⚠️ 높음

# Network
network_bytes_in_rate: 100 Mbps / 1000 Mbps = 10%

# Saturation
cpu_queue_length: 2  # CPU 대기 프로세스
disk_io_wait_percent: 5%  # I/O 대기
```

---

## SLI와 SLO

### "SLI와 SLO는 무엇이 다른가?"

**SLI (Service Level Indicator)는 실제 서비스 품질을 측정하는 지표, SLO (Service Level Objective)는 목표 수준을 정의하는 정책**다.

```text
SLI (Service Level Indicator): 측정 지표
- "99.9%의 요청이 200ms 이내에 응답했다"
- "99.99%의 시간 서비스가 가용했다"
- 실제 데이터

SLO (Service Level Objective): 목표값
- "99.9% 가용성을 유지하겠다"
- "평균 응답 시간 100ms 이하를 목표로 한다"
- 약속, 목표

SLA (Service Level Agreement): 계약
- "99.9% 가용성 보장, 위반 시 환불"
- 사업 계약
```

### SLI/SLO 정의 예시

```yaml
서비스: "주문 처리 API"

SLO (목표):
- 가용성: 99.9% (월 43분 다운타임 허용)
- 응답 시간: p99 < 500ms
- 에러율: < 0.1%

SLI (측정):
- 가용성 = (성공한 요청) / (전체 요청)
- 응답 시간 = 응답 시간 히스토그램의 p99
- 에러율 = (5xx 상태 코드 요청) / (전체 요청)

현재 상태:
- 가용성: 99.95% ✓ (목표 초과)
- 응답 시간 p99: 250ms ✓ (목표 달성)
- 에러율: 0.05% ✓ (목표 달성)
```

### Error Budget

**SLO와 실제의 차이를 "Error Budget"이라 하며, 배포 전에 얼마나 실패할 수 있는지 정량화**한다.

```text
SLO: 99.9% (월 1000분 중 실패 허용: 1000 * (1 - 0.999) = 1분)

현재 누적:
- 1주: 0분 사용 (100% 달성)
- 2주: 5초 사용 (계획 초과 배포로 다운)
- 3주: 30초 사용 (긴급 패치로 다운)

남은 Error Budget:
1분 - 35초 = 25초 (이 달 남은 실패 허용 시간)

의사결정:
- 남은 budget이 충분 → 자신감 있게 배포
- budget이 부족 → 신중하게 배포 (이미 초과 위험)
- budget 없음 → 배포 금지, 안정성 개선 우선
```

---

## Cardinality

### "cardinality가 높으면 왜 문제가 되는가?"

**Cardinality는 Label 조합의 개수이며, 높을수록 메모리/비용 증가**한다.

```text
낮은 Cardinality:
http_requests_total{method="GET", status="200"}
http_requests_total{method="GET", status="500"}
http_requests_total{method="POST", status="200"}
→ 3개 조합

높은 Cardinality (위험):
http_requests_total{method="GET", status="200", user_id="123"}
http_requests_total{method="GET", status="200", user_id="456"}
http_requests_total{method="GET", status="200", user_id="789"}
... (사용자 수만큼)
→ 사용자 수 만큼 증가!
```

### Cardinality 문제

```text
사용자별 메트릭:
- 사용자 100명 → 100개 시계열
- 사용자 1만 명 → 1만 개 시계열
- 사용자 100만 명 → 100만 개 시계열 ← 메모리 부족!

비용:
- 메트릭 저장소 비용 (시계열 개수 기준)
- 쿼리 성능 저하
- 캐시 비효율
```

### 높은 Cardinality 피하기

```python
# ❌ 나쁜 예 (user_id가 Cardinality 폭발)
from prometheus_client import Counter

request_by_user = Counter(
    'requests_by_user',
    'Requests per user',
    ['user_id']  # 사용자가 많으면 문제!
)

# ✓ 좋은 예 (고정된 Label만 사용)
request_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'status']  # 고정된 값만 사용
)

# ✓ 별도의 조회 방식 사용
user_request_count = Counter(
    'user_requests_total',
    'Total requests by user',
    ['tier']  # gold, silver, bronze 같은 카테고리만
)
```

### Label 설계 원칙

```text
✓ 사용할 수 있는 Label:
- method: GET, POST, PUT, DELETE (4-10개)
- status: 200, 400, 500 (10개 이내)
- endpoint: /api/users, /api/orders (수십 개)
- tier: gold, silver, bronze (수개)
- region: us-east, eu-west (수개)

❌ 사용하면 안 되는 Label:
- user_id (수백만 개)
- request_id (무한)
- timestamp (무한)
- trace_id (무한)
```

---

## Metric 설계

### 좋은 Metric 설계

```text
1. 비즈니스 지표
   - 주문 수 (매분)
   - 매출 (매시간)
   - 사용자 가입 (매일)

2. 기술 지표 (RED)
   - Request rate
   - Error rate
   - Duration (p50, p95, p99)

3. 자원 지표 (USE)
   - CPU, Memory, Disk 사용률
   - Network 사용
   - I/O 대기

4. 도메인 특화 지표
   - 데이터베이스: 쿼리 시간, slow query 수
   - 캐시: hit rate, miss rate
   - 메시지 큐: 큐 깊이, consumer lag
```

### Naming 규칙

```
<namespace>_<subsystem>_<name>_<unit>

예시:
http_request_duration_seconds
├─ http: namespace
├─ request: subsystem
├─ duration: name
└─ seconds: unit

규칙:
- 소문자, 언더스코어로 구분
- 단위는 suffix로 (seconds, bytes, percent)
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Counter | 계속 증가하는 값 (총 요청 수) |
| Gauge | 현재값 (메모리 사용, CPU) |
| Histogram | 값의 분포 (응답 시간 분포) |
| RED | Request Rate, Error Rate, Duration |
| USE | Utilization, Saturation, Errors |
| SLI | 실제 서비스 품질 측정 지표 |
| SLO | 목표 서비스 수준 |
| Cardinality | Label 조합 개수 (높으면 문제) |

---

## Related Notes

- [Logging](../logging/overview.md)
- [Prometheus](../prometheus/overview.md)
