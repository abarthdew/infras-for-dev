# Datadog

## 목차
- [Datadog의 역할](#datadog의-역할)
- [Datadog Agent](#datadog-agent)
- [Metrics](#metrics)
- [Logs](#logs)
- [Traces와 APM](#traces와-apm)
- [Monitor](#monitor)
- [Integration](#integration)
- [비용 및 고려사항](#비용-및-고려사항)

---

## Datadog의 역할

**Datadog은 metrics, logs, traces를 한 플랫폼에서 수집하고 분석하는 클라우드 기반 SaaS 관측성 도구**다. Prometheus, ELK, Jaeger 같은 오픈소스 도구를 따로 호스팅하지 않고, 관리형 서비스로 제공한다.

```text
Datadog Agent (설치)
    ↓ (수집)
┌─ Metrics
├─ Logs
├─ Traces
└─ Infrastructure metrics
    ↓
Datadog Platform (클라우드)
    ├─ Dashboard
    ├─ Monitor
    ├─ Alert
    ├─ Analytics
    └─ Integration
    ↓
사용자 (웹 UI)
```

### Datadog vs 오픈소스 도구

```text
Prometheus + ELK + Jaeger:
장점: 자유도, 비용 (자체 호스팅 가능)
단점: 운영 복잡, 통합 어려움, 자체 관리 필요

Datadog:
장점: 통합 플랫폼, 관리형 서비스, 고급 기능
단점: 높은 비용, vendor lock-in, 데이터 주권 고민
```

---

## Datadog Agent

### Agent 설치

```bash
# macOS
brew install datadog-agent

# Linux
DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=<API_KEY> DD_SITE="datadoghq.com" bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_agent.sh)"

# Docker
docker run -d \
  --name datadog \
  -e DD_API_KEY=<API_KEY> \
  -e DD_SITE=datadoghq.com \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  datadog/agent:latest
```

### Agent 설정

```yaml
# /etc/datadog-agent/datadog.yaml
api_key: <API_KEY>

logs:
  enabled: true

apm:
  enabled: true
  bind_host: 0.0.0.0
  receiver_port: 8126

integrations:
  docker:
    enabled: true
  prometheus:
    enabled: true
```

---

## Metrics

### Datadog Metrics 수집

```text
Integration: 자동 수집
├─ Docker 메트릭
├─ Kubernetes 메트릭
├─ MySQL, PostgreSQL
├─ Redis, Elasticsearch
└─ AWS CloudWatch

Custom Metrics: 애플리케이션에서 송신
├─ 비즈니스 메트릭 (주문 수, 매출)
├─ 애플리케이션 메트릭
└─ 커스텀 게이지/카운터
```

### Custom Metric 보내기

```python
from datadog import initialize, api
import time

options = {
    'api_key': '<API_KEY>',
    'app_key': '<APP_KEY>'
}

initialize(**options)

# 현재 값 보내기 (Gauge)
api.Metric.send(
    metric='my_app.user_count',
    points=150,
    tags=['environment:prod', 'service:api']
)

# 시간대 값 보내기
api.Metric.send(
    metric='my_app.orders_total',
    points=[(int(time.time()), 5), (int(time.time()) + 60, 8)],
    tags=['environment:prod']
)
```

---

## Logs

### Log 수집

```text
파일 기반:
- /var/log/app.log → Agent → Datadog
- JSON 파싱 자동
- 필드 추출 자동

Integration 기반:
- Docker logs
- Kubernetes logs
- AWS CloudWatch

API 기반:
- 애플리케이션에서 직접 송신
```

### Log 설정

```yaml
# datadog.yaml에 로그 설정
logs:
  enabled: true

# /etc/datadog-agent/conf.d/app.d/conf.yaml
logs:
  - type: file
    path: /var/log/app.log
    service: my_app
    source: python
    tags:
      - environment:prod
```

---

## Traces와 APM

### "APM trace는 log와 무엇이 다른가?"

**APM Trace는 요청의 구조화된 흐름 (span 계층), Log는 자유형식 텍스트 메시지**다.

```text
Log:
2026-05-23T10:15:30 [INFO] Order created for user 123
2026-05-23T10:15:31 [INFO] Processing payment
2026-05-23T10:15:32 [ERROR] Payment failed: timeout

문제: 연결이 없음, 시간만으로 추적

APM Trace:
trace_id: abc123
├─ span: create_order (100ms)
│  └─ log: "Order created for user 123"
├─ span: process_payment (900ms)
│  └─ log: "Processing payment"
│  └─ log: "Payment failed: timeout" → 여기서 실패!
└─ span: handle_error (20ms)

장점: 구조화, 계층, 원인 파악 쉬움
```

### APM 자동 계측

```python
# Flask 앱에서 자동으로 trace 수집
from ddtrace import patch_all

patch_all()  # 모든 라이브러리 자동 계측

from flask import Flask
app = Flask(__name__)

@app.route('/orders/<order_id>')
def get_order(order_id):
    # 자동으로 span 생성
    # - HTTP handler
    # - DB 쿼리
    # - 외부 API 호출
    return {"status": "ok"}
```

### Datadog APM 대시보드

```text
Service Map:
- 서비스 간 의존성 시각화
- 각 서비스 상태 표시

Latency Heatmap:
- 응답 시간 분포
- p50, p95, p99 표시

Trace Search:
- 특정 조건의 trace 검색
- 에러 trace 필터링
```

---

## Monitor

### "monitor threshold는 어떻게 잡아야 하는가?"

**Monitor threshold는 False Positive와 False Negative의 균형을 맞춰야 한다.**

```text
너무 엄격한 threshold (False Positive):
- threshold: CPU > 50%
- 정상 부하도 alert
- Slack에 스팸
- 운영팀이 알림 무시

너무 느슨한 threshold (False Negative):
- threshold: CPU > 95%
- 문제를 늦게 발견
- 이미 사용자에게 영향

적절한 threshold:
- 일반적 부하: 50%
- 경고: 75% (대비 필요)
- 심각: 90% (즉각 조치)
```

### Monitor 유형

```text
Metric Monitor:
- CPU > 80% for 5 min
- Memory > 70%
- Disk > 85%

Log Monitor:
- ERROR 로그 > 10/min
- 특정 에러 메시지 감지

APM Monitor:
- Latency p99 > 500ms
- Error rate > 1%
- Apdex score < 0.95

Synthetic Monitor:
- API endpoint 응답성
- 페이지 로딩 시간
- 전국 여러 위치에서 테스트
```

### Monitor 설정

```yaml
# Monitor 생성
name: "High CPU Usage"
type: metric alert
query: avg:system.cpu.user{*} > 0.8
for: "5m"
notify:
  - "@slack-#alerts"
  - "@pagerduty"
tags:
  - environment:prod
```

---

## Integration

### 주요 Integration

```text
클라우드:
- AWS (EC2, RDS, S3, CloudWatch)
- GCP (Compute Engine, Cloud SQL)
- Azure (VMs, SQL Database)

데이터베이스:
- MySQL, PostgreSQL
- MongoDB, Redis
- Elasticsearch

애플리케이션:
- Docker, Kubernetes
- Jenkins, GitLab CI
- PagerDuty, Slack

모니터링:
- Prometheus
- New Relic
- Splunk
```

### Integration 설정

```bash
# AWS Integration
1. Datadog Console → Integration → AWS
2. "Add AWS Account"
3. IAM Role 생성 (Datadog에게 권한 부여)
4. 선택할 metrics 지정
5. 완료

# 자동으로 AWS metrics 수집 시작
```

---

## "tag 설계는 왜 중요한가?"

**Tag는 metrics를 필터링하고 그룹화하는 방식이므로, 설계가 좋아야 쿼리와 대시보드가 효율적**이다.

```text
나쁜 tag 설계:
metric: request_latency
tags:
  - env=prod
  - host=server-xyz-abc-123
  - app_version=1.2.3.4
  - user_id=123456
  - transaction_type=payment
  - ...

문제: cardinality 폭발
- user_id 추가 → 백만 개 태그
- 저장 비용 증가
- 쿼리 성능 저하

좋은 tag 설계:
tags:
  - environment:prod
  - service:payment
  - region:us-east
  - tier:premium  # 사용자 카테고리
  - version:1.2.3
```

### Tag 명명 규칙

```text
key:value 형식
- environment:prod, staging, dev
- service:api, web, worker
- region:us-east, eu-west
- tier:gold, silver, bronze
- version:1.2.3
- deployment:blue, green

Datadog 예약 tag:
- host: hostname
- service: service name
- version: app version
- env: environment

금지:
- 고 cardinality (user_id, request_id)
- 무의미한 값 (true, false)
- 과도한 태그 (> 10개/metric)
```

---

## 비용 및 고려사항

### "vendor lock-in은 어떤 고민을 만든다?"

**Datadog 종속성이 높아지면, 나중에 다른 도구로 마이그레이션하기 어려워질 수 있다.**

```text
종속성 증가:
- Dashboard, Monitor를 Datadog에만 구축
- 데이터 형식이 Datadog 특화
- 고급 기능 (APM, 분석) 의존도 높음
- 직원들이 Datadog UI에만 익숙

마이그레이션 어려움:
- 기존 대시보드 재구축 필요
- Monitor 규칙 변환 필요
- 데이터 내보내기 어려움 (30일 제한)
- 새 도구 학습 필요
```

### 비용 관리

```text
Datadog 청구 기준:
- Metrics: 호스트 수 (Host: $15/월)
- Logs: GB 수집량 (Logs: $0.10/GB)
- Traces: 수집된 span 수 (APM: $0.10/million spans)
- Synthetic: 테스트 실행 수

비용 절감:
1. 필요 없는 metrics 수집 비활성화
2. Log sampling (모든 로그 대신 일부만)
3. Trace sampling (모든 trace 대신 일부만)
4. 오래된 메트릭 삭제
5. 비용 예산 설정 및 모니터링
```

### Datadog vs 오픈소스 선택

```text
Datadog 선택:
- 관리형 서비스 필요 (운영 부담 줄이기)
- 고급 기능 필요 (APM, 이상 탐지)
- 엔터프라이즈 지원 필요
- 비용에 여유 있음

오픈소스 선택:
- 비용 절감 중시
- 데이터 주권 중요 (on-prem 필요)
- 종속성 최소화 필요
- 운영 인력 충분
```

---

## 실전 예시: 성능 저하 진단

```
User Report: 서비스가 느려졌다

Step 1: APM Trace 조회
- 느린 trace 확인
- 어느 단계에서 병목? → Database query (2초)

Step 2: 관련 Log 확인
- DB query 로그
- "Slow query: SELECT ... took 2000ms"

Step 3: Host Metric 확인
- DB 서버 CPU: 95%
- DB 서버 Memory: 85%
- DB 서버 Disk I/O: 높음

원인 분석:
- DB 서버 과부하
- 캐시 미스율 증가? → Redis metrics 확인
- 쿼리 비효율? → 실행 계획 분석

해결책:
1. DB 쿼리 최적화
2. 캐시 전략 개선
3. 자동 스케일링 규칙 조정
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Agent | 호스트에서 데이터 수집 및 전송 |
| Metrics | 시계열 수치 데이터 |
| Logs | 텍스트 로그 메시지 |
| Traces/APM | 요청 흐름 추적 |
| Monitor | 임계값 기반 알림 |
| Integration | 외부 시스템 연동 |
| Tag | 메트릭 분류 및 필터링 |

---

## Related Notes

- [Metrics](../metrics/overview.md)
- [Logging](../logging/overview.md)
- [Prometheus](../prometheus/overview.md)
