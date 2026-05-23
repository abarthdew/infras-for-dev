# Tracing

## 목차
- [Tracing의 역할](#tracing의-역할)
- [Trace와 Span](#trace와-span)
- [Context Propagation](#context-propagation)
- [Distributed Tracing](#distributed-tracing)
- [Sampling](#sampling)
- [Tracing 구현](#tracing-구현)

---

## Tracing의 역할

**Tracing은 하나의 사용자 요청이 여러 마이크로서비스, 데이터베이스, 캐시를 지나가며 남기는 흐름을 추적하는 관측성 기술**이다. 복잡한 분산 시스템에서 병목과 지연을 식별하는 데 필수적이다.

```text
사용자 요청
    ↓ (trace_id: abc123으로 시작)
API Gateway (span 1)
    ↓
User Service (span 2)
    ├─ Database (span 3)
    └─ Cache (span 4)
    ↓
Order Service (span 5)
    ├─ Database (span 6)
    └─ Kafka (span 7)
    ↓
Payment Service (span 8)
    └─ Payment Gateway (span 9)

→ 전체 흐름이 명확하게 보임
```

### Observability 삼각형

```text
Logs (개별 사건)
- "에러 발생함"
- "DB 쿼리 실패"

Metrics (집계 통계)
- "에러율 5%"
- "p99 latency 500ms"

Traces (요청 흐름)
- "이 요청은 왜 500ms 걸렸나?"
- "어디서 시간을 낭비했나?"

→ 3가지를 함께 봐야 완전한 이해
```

---

## Trace와 Span

### "trace와 log는 무엇이 다른가?"

**Log는 개별 이벤트의 상세 정보, Trace는 요청의 전체 여정을 구조화한 기록**이다.

```text
Log (개별 사건):
"2026-05-23T10:15:30 [ERROR] Failed to fetch user: connection timeout"

Log만으로는:
- 어디서 실패했나? (service A? B? C?)
- 그 후 무엇이 실패했나?
- 누가 이 요청을 했나?
→ 전체 흐름을 파악하기 어려움

Trace (요청의 여정):
trace_id=abc123
├─ span: API handler 100ms
│  ├─ log: "Request received"
│  └─ log: "Validating input"
├─ span: User Service 50ms
│  ├─ log: "Fetching user"
│  └─ log: "ERROR: connection timeout" ← 여기서 실패!
└─ span: fallback 5ms
   └─ log: "Using cached user"

→ 전체 흐름이 명확함
```

### "span은 어떤 단위를 나타내는가?"

**Span은 한 작업 단위의 시간 구간**이다. 함수 호출, HTTP 요청, 데이터베이스 쿼리 등이 span이 될 수 있다.

```text
Span 구성:
├─ trace_id: 이 span이 속한 trace (abc123)
├─ span_id: 이 span의 식별자 (span-1)
├─ parent_span_id: 부모 span (span-0)
├─ operation_name: "HTTP GET /users"
├─ start_time: 2026-05-23T10:15:30.000Z
├─ duration: 50ms (또는 end_time)
├─ tags:
│  ├─ http.method: "GET"
│  ├─ http.status_code: 200
│  └─ service.name: "user-service"
└─ logs: [
     {"timestamp": "...", "message": "Query executed"},
     {"timestamp": "...", "message": "Response sent"}
   ]
```

### Span 계층 구조

```text
trace_id: abc123
├─ span-0 (API Gateway) - 100ms
│  ├─ span-1 (User Service) - 50ms
│  │  ├─ span-2 (DB query) - 40ms
│  │  └─ span-3 (Cache) - 2ms
│  └─ span-4 (Order Service) - 30ms
│     └─ span-5 (Message Queue) - 25ms
└─ span-6 (Load Balancer) - 5ms
```

---

## Context Propagation

### "context propagation은 왜 필요한가?"

**Context Propagation은 trace_id와 span_id를 서비스 간에 전달해서 분산된 span들을 연결**한다.

```text
문제 (propagation 없이):
Service A에서 Service B로 HTTP 요청
- Service A: trace_id=abc123, span-1
- Service B에서 새로 trace_id=xyz789 생성
- 두 span이 서로 연결 안 됨!

해결 (propagation 사용):
Service A에서 Service B로 HTTP 요청 시
Headers에 추가:
- X-Trace-Id: abc123
- X-Parent-Span-Id: span-1
- X-Span-Id: span-2 (새 span)

Service B에서:
- 받은 trace_id=abc123 사용
- parent_span_id=span-1로 설정
- 새 span-2 생성
→ 연결됨!
```

### Context 전파 방식

```text
HTTP Headers (가장 일반적):
GET /order
Headers:
  X-Trace-Id: abc123
  X-Parent-Span-Id: span-1
  X-Span-Id: span-2

Message Queue (Kafka, RabbitMQ):
{
  "user_id": 123,
  "trace_id": "abc123",
  "parent_span_id": "span-1"
}

gRPC Metadata:
metadata {
  key: "x-trace-id"
  value: "abc123"
}
```

### 구현

```python
from opentelemetry import trace, propagate
from opentelemetry.propagators.jaeger import JaegerPropagator

# 수신 측 (Service B)
# HTTP headers에서 context 추출
parent_context = propagate.extract("http", request.headers)

# 새 span 생성 (parent context 지정)
with tracer.start_as_current_span("service_b_operation", context=parent_context) as span:
    # 작업 수행
    pass

# 송신 측 (Service A에서 B로 호출)
# 현재 context를 headers에 주입
headers = {}
propagate.inject("http", headers)
# headers에 x-trace-id, x-parent-span-id 등이 추가됨

response = requests.get("http://service-b/api", headers=headers)
```

---

## Distributed Tracing

### 분산 추적 플랫폼

```text
Jaeger (오픈소스):
- 자체 호스팅
- 우수한 성능
- 커뮤니티 지원

Zipkin (오픈소스):
- 자체 호스팅
- 사용자 친화적 UI
- 경량

DataDog:
- 클라우드 서비스
- 풍부한 기능
- 높은 비용

AWS X-Ray:
- AWS 네이티브
- CloudWatch 통합
- AWS 환경에서 편리

Lightstep:
- 클라우드 서비스
- 실시간 분석
- 높은 비용
```

### Jaeger 아키텍처

```text
Application (Jaeger Client 라이브러리)
    ↓ (traces 수집)
Jaeger Agent (localhost:6831)
    ↓
Jaeger Collector
    ↓
Elasticsearch (저장)
    ↓
Jaeger UI (조회, 분석)
```

### Jaeger 설치 및 사용

```bash
# Docker로 Jaeger 실행
docker run -d --name jaeger \
  -p 6831:6831/udp \
  -p 16686:16686 \
  jaegertracing/all-in-one

# 접속
http://localhost:16686
```

---

## Sampling

### "sampling은 정확도와 비용 사이에서 어떤 선택인가?"

**Sampling은 모든 trace를 기록하지 않고 일부만 기록해서 비용을 절감하면서도 충분한 정보를 유지하는 전략**이다.

```text
전체 기록:
- Request: 1M/일
- 모든 trace 기록 → 1M traces
- 저장 비용: 높음
- 분석: 느림

50% Sampling:
- Request: 1M/일
- 50%만 기록 → 500K traces
- 저장 비용: 절반
- 손실: 500K traces 분석 불가

문제:
- 드문 에러를 놓칠 수 있음
- 완전한 그림이 아님
```

### Sampling 전략

```text
Static Sampling (고정 비율):
- 모든 trace의 1%만 기록
- 단순하지만 비효율
- 중요한 요청도 1%만 기록될 수 있음

Adaptive Sampling (적응형):
- 정상 요청: 0.1% (불필요)
- 느린 요청: 100% (분석 필요)
- 에러 요청: 100% (원인 파악 필요)

Head-based Sampling:
- 요청 시작 시 샘플 여부 결정
- Service A에서 결정 → 모든 downstream에 전파
- 장점: 일관성
- 단점: 요청이 올 때까지 중요도 모름

Tail-based Sampling:
- 요청 완료 후 샘플 여부 결정
- 실제 성능 기반 샘플링
- 장점: 정확
- 단점: 모든 trace 일단 저장 (비용)
```

### Sampling 구현

```python
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.trace.sampler import TraceIdRatioBased

# 50% sampling
sampler = TraceIdRatioBased(0.5)

tracer_provider = TracerProvider(sampler=sampler)

# 또는 환경 변수로 조정
import os
ratio = float(os.getenv('SAMPLING_RATIO', '0.1'))
sampler = TraceIdRatioBased(ratio)
```

---

## Tracing 구현

### OpenTelemetry (표준)

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger import JaegerExporter

# Jaeger exporter 설정
jaeger_exporter = JaegerExporter(
    agent_host_name='localhost',
    agent_port=6831,
)

# Tracer 설정
tracer_provider = TracerProvider()
tracer_provider.add_span_processor(BatchSpanProcessor(jaeger_exporter))
trace.set_tracer_provider(tracer_provider)

# 사용
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("my_operation") as span:
    span.set_attribute("user_id", 123)
    span.set_attribute("http.method", "GET")
    
    # 작업 수행
    result = some_operation()
    
    span.set_attribute("http.status_code", 200)
```

### 자동 계측 (Instrumentation)

```python
# Django 자동 계측
from opentelemetry.instrumentation.django import DjangoInstrumentor

DjangoInstrumentor().instrument()

# 이제 모든 Django 요청이 자동으로 span 생성
# - HTTP handler
# - 템플릿 렌더링
# - ORM 쿼리
```

---

## 실전 분석 예시

```
조회 쿼리: trace_id=abc123

Timeline View:
├─ API Gateway (0-100ms)
├─ User Service (10-60ms) - 느림!
│  ├─ DB query (15-55ms) - 40ms 소요
│  └─ Cache lookup (12-14ms) - 2ms
└─ Order Service (65-100ms)

분석:
1. User Service가 느린 이유: DB query 40ms
2. 왜 DB가 느린가?
   - n+1 query? → 로그 확인
   - 인덱스 없음? → DB 실행 계획 확인
   - 네트워크 지연? → 지연 시간 확인
3. 개선안: 캐시 활용 (2ms로 줄어들 수 있음)
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Trace | 한 요청의 전체 여정 |
| Span | Trace 내 한 작업 단위 |
| Context Propagation | 서비스 간 trace 정보 전달 |
| Distributed Tracing | 분산 시스템의 요청 추적 |
| Sampling | 비용/정확도 균형 (일부만 기록) |

---

## Related Notes

- [Logging](../logging/overview.md)
- [Metrics](../metrics/overview.md)
