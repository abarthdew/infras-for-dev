# Logging

## 목차
- [Logging의 역할](#logging의-역할)
- [Log Level](#log-level)
- [Structured Logging](#structured-logging)
- [Correlation ID](#correlation-id)
- [Log Aggregation](#log-aggregation)
- [Retention](#retention)
- [보안과 민감 정보](#보안과-민감-정보)

---

## Logging의 역할

**로그는 시스템에서 발생한 사건을 시간 순서로 남긴 기록**이다. 장애 분석, 감사, 디버깅, 성능 모니터링에 필수적이지만 너무 많으면 비용과 소음이 증가한다.

```text
Application
    ↓ (이벤트 발생)
Logging
    ↓ (시간 순서로 기록)
Log File (Local)
    ↓ (수집)
Collector (Fluentd, Logstash)
    ↓ (저장)
Storage (Elasticsearch, S3)
    ↓ (검색/분석)
Logging Platform (Kibana, Splunk)
```

### 로그가 필요한 이유

```text
장애 분석:
- "왜 주문 처리가 실패했는가?"
- 로그에서 ERROR부터 역추적

성능 모니터링:
- "어느 함수가 느린가?"
- execution time 로그로 추적

감사(Audit):
- "누가 언제 이 데이터를 수정했는가?"
- 모든 주요 작업 기록

보안:
- "비정상 접근이 있었는가?"
- 로그인 실패, 권한 위반 기록
```

---

## Log Level

### "DEBUG, INFO, WARN, ERROR는 어떻게 구분하는가?"

**Log Level은 로그의 심각도를 나타내며, 어떤 수준 이상의 로그만 출력할지 결정**한다.

```text
DEBUG
│ 개발자 디버깅용 상세 정보
│ 예: 함수 진입/종료, 변수값, SQL 쿼리
│ 프로덕션에서는 꺼짐 (성능 영향)
│
INFO
│ 애플리케이션 정상 동작 정보
│ 예: 요청 수신, 처리 완료, 서버 시작
│ 프로덕션에서 기본 레벨
│
WARN
│ 문제가 될 가능성 있음
│ 예: deprecated API 사용, 권장되지 않는 설정, 높은 메모리 사용
│ 주의 필요 (곧 에러가 될 수 있음)
│
ERROR
│ 에러 발생, 작업 실패
│ 예: 데이터베이스 연결 실패, 유효성 검증 실패
│ 즉시 대응 필요
│
FATAL/CRITICAL
  애플리케이션 자체가 동작 불가
  예: 시스템 시작 실패, 메모리 부족
  긴급 대응 필수
```

### Level별 예시

```python
import logging

logger = logging.getLogger(__name__)

# DEBUG: 상세한 흐름 정보
logger.debug(f"Processing order {order_id} from user {user_id}")

# INFO: 정상 동작 정보
logger.info(f"Order {order_id} created successfully")

# WARN: 주의 필요
logger.warning(f"Slow query detected: {query_time}ms > 1000ms threshold")

# ERROR: 에러 (작업 실패하지만 앱은 계속)
logger.error(f"Failed to send email to {email}: {error}")

# CRITICAL: 심각한 에러 (앱 중단)
logger.critical(f"Database connection lost, cannot continue")
```

### Log Level 설정

```yaml
# production
log_level: INFO
# DEBUG, INFO, WARN, ERROR, CRITICAL만 기록

# staging
log_level: DEBUG
# 모든 레벨 기록

# local development
log_level: DEBUG
# 디버깅 정보 포함
```

### Level별 저장 비용

```text
DEBUG 포함: 로그 용량 10배 증가, 비용 10배
INFO 기본: 1GB/일 (기준)
WARN/ERROR만: 100MB/일 (10% 비용)

→ 환경별로 다르게 설정 필수
```

---

## Structured Logging

### "structured logging은 왜 검색에 유리한가?"

**Structured Logging은 로그를 JSON 같은 정형화된 형식으로 저장해서 필드별로 검색/필터링 가능**하게 한다.

```text
Text Logging (비정형):
"2026-05-23T10:15:30 ERROR Failed to create order for user 12345"
→ 문자열 검색만 가능 (느림, 부정확)

Structured Logging (정형):
{
  "timestamp": "2026-05-23T10:15:30",
  "level": "ERROR",
  "action": "create_order",
  "user_id": 12345,
  "error_code": "INVALID_PAYMENT",
  "message": "Failed to create order"
}
→ 필드별 검색 가능 (빠름, 정확)
```

### 검색 예시

```sql
-- Text 검색 (느리고 부정확)
grep "ERROR.*order" logs.txt

-- Structured 검색 (빠르고 정확)
SELECT * FROM logs WHERE level='ERROR' AND action='create_order'
SELECT * FROM logs WHERE user_id=12345 AND timestamp > '2026-05-23'
SELECT COUNT(*) FROM logs WHERE error_code='INVALID_PAYMENT'
```

### Structured Logging 구현

```python
import logging
import json
from datetime import datetime

class StructuredFormatter(logging.Formatter):
    def format(self, record):
        log_dict = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
        }
        
        # 추가 필드
        if hasattr(record, 'user_id'):
            log_dict['user_id'] = record.user_id
        if hasattr(record, 'request_id'):
            log_dict['request_id'] = record.request_id
        
        return json.dumps(log_dict)

# 사용
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
formatter = StructuredFormatter()
handler.setFormatter(formatter)
logger.addHandler(handler)

# 로깅 시 추가 정보
logger = logging.LoggerAdapter(logger, {'user_id': 12345, 'request_id': 'abc-123'})
logger.info("Order created")
```

### Structured Logging의 이점

```text
검색 속도:
- Text: 초 단위
- Structured: 밀리초 단위

정확성:
- Text: "order" 검색 → 모든 "order" 포함 단어 매칭
- Structured: action='order_created' 필드만 매칭

분석:
- Text: 수동 파싱 필요
- Structured: 바로 집계 가능 (user별 에러율, 시간대별 성능 등)
```

---

## Correlation ID

### "correlation id는 분산 시스템에서 왜 필요한가?"

**Correlation ID는 하나의 사용자 요청이 여러 서비스를 거쳐가며 남기는 로그들을 연결하는 식별자**다.

```text
사용자 요청
    ↓ API Gateway (correlation_id: req-abc123)
    
    ├→ User Service (로그: correlation_id=req-abc123)
    │   ├→ Database (로그: correlation_id=req-abc123)
    │   └→ Cache (로그: correlation_id=req-abc123)
    │
    ├→ Order Service (로그: correlation_id=req-abc123)
    │   ├→ Database (로그: correlation_id=req-abc123)
    │   └→ Kafka (로그: correlation_id=req-abc123)
    │
    └→ Payment Service (로그: correlation_id=req-abc123)
        ├→ Payment Gateway (로그: correlation_id=req-abc123)
        └→ Database (로그: correlation_id=req-abc123)

→ correlation_id=req-abc123로 모든 로그 추적 가능
```

### 없으면 어떻게 되는가?

```text
문제: "주문이 실패했다고 하는데 왜?"

Correlation ID 없이:
- 로그 시간으로 찾기 (부정확, 동시 요청 구분 불가)
- 여러 서비스의 로그를 연결할 수 없음
- 전체 흐름을 따라가기 어려움

Correlation ID 사용:
grep "correlation_id=req-abc123" all-logs.txt
→ 전체 흐름이 명확하게 보임
```

### 구현

```python
from uuid import uuid4
import logging

class RequestContextFilter(logging.Filter):
    def filter(self, record):
        record.correlation_id = getattr(request_context, 'correlation_id', 'none')
        return True

# HTTP 요청 처리 시
from flask import Flask, request
import uuid

app = Flask(__name__)

@app.before_request
def before_request():
    # 기존 header에 있으면 사용, 없으면 생성
    correlation_id = request.headers.get('X-Correlation-ID', str(uuid4()))
    request.correlation_id = correlation_id

@app.after_request
def after_request(response):
    # Response에도 correlation ID 포함 (클라이언트가 참조 가능)
    response.headers['X-Correlation-ID'] = request.correlation_id
    return response
```

### Correlation ID 전파

```python
# Service A에서 Service B를 호출할 때
import requests

correlation_id = request.correlation_id

# Service B로 correlation_id 전달
response = requests.get(
    'http://service-b/endpoint',
    headers={'X-Correlation-ID': correlation_id}
)
```

---

## Log Aggregation

### Log Aggregation이란?

**여러 서버/서비스의 로그를 한 곳에 모아서 중앙에서 검색하고 분석**한다.

```text
Server 1 (로그 파일: /var/log/app.log)
Server 2 (로그 파일: /var/log/app.log)
Server 3 (로그 파일: /var/log/app.log)
    ↓ (수집)
Log Collector (Fluentd, Logstash, Beats)
    ↓ (전송)
Central Storage (Elasticsearch, Splunk, CloudWatch)
    ↓ (검색)
UI (Kibana, Splunk, Datadog)
```

### 필요한 이유

```text
로그 aggregation 없이:
- 각 서버에 SSH 접속
- tail -f로 실시간 확인
- 여러 서버 동시 모니터링 불가
- 과거 로그 검색 불가

로그 aggregation 사용:
- 중앙 UI에서 모든 로그 검색
- 실시간 대시보드
- 여러 서버의 로그 관계 분석
- 자동 알림 설정
```

### 구현 예시 (Elasticsearch + Logstash + Kibana)

```bash
# 1. Logstash 설정 (/etc/logstash/conf.d/app.conf)
input {
  file {
    path => "/var/log/app.log"
    start_position => "beginning"
  }
}

filter {
  json {
    source => "message"
  }
  date {
    match => [ "timestamp", "ISO8601" ]
    target => "@timestamp"
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "app-%{+YYYY.MM.dd}"
  }
}

# 2. 실행
logstash -f /etc/logstash/conf.d/app.conf

# 3. Kibana에서 검색
# URL: http://localhost:5601
# Index: app-*
# 검색: level=ERROR AND correlation_id=req-abc123
```

---

## Retention

### Retention이란?

**로그를 어떤 기간 동안 보관할지 결정하는 정책**이다.

```text
Retention 기간별 비용:
7일: 최소 비용 (최근 문제만)
30일: 중간 비용 (1개월 분석)
90일: 높은 비용 (분기별 분석)
1년: 매우 높은 비용 (감사, 규제 준수)

→ 규제 요구사항과 비용을 고려해 결정
```

### 환경별 Retention

```yaml
Development:
  retention: 7 days  # 최근 이슈만 추적
  cost: 낮음

Staging:
  retention: 30 days  # 배포 전 테스트 결과 보관
  cost: 중간

Production:
  retention: 90 days  # 장애 분석, 성능 추적
  cost: 높음

Compliance (금융/의료):
  retention: 1-7 years  # 규제 요구사항
  cost: 매우 높음
```

### Retention 구현

```yaml
# Elasticsearch Index Lifecycle Management
PUT _ilm/policy/app-logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0d",
        "actions": {
          "rollover": {
            "max_size": "10gb",
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "3d",
        "actions": {
          "set_priority": {
            "priority": 50
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

## 보안과 민감 정보

### "민감 정보는 로그에 왜 남기면 안 되는가?"

**로그는 다양한 사람이 접근하고, 보안 위협에 노출될 수 있으므로 민감 정보를 기록하면 데이터 유출 위험**이 높아진다.

```text
위험한 로그:
logger.info(f"User {user_id} logged in with password {password}")
→ 누가 로그 파일/UI 봤을 때 비밀번호 노출!

안전한 로그:
logger.info(f"User {user_id} logged in")
→ 비밀번호는 기록하지 않음
```

### 민감 정보 예시

```text
절대 기록하면 안 됨:
- 비밀번호
- API 키, 토큰
- 신용카드 번호
- SSN, 주민등록번호
- 개인 이메일, 휴대폰
- 의료 정보

기록해도 되지만 마스킹:
- 이메일: user***@example.com
- 카드번호: ****-****-****-1234
- IP: 203.0.113.***
```

### 마스킹 구현

```python
import re
import logging

class MaskingFilter(logging.Filter):
    def filter(self, record):
        # 신용카드 번호 마스킹
        record.msg = re.sub(
            r'\b\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}\b',
            '****-****-****-****',
            record.msg
        )
        
        # API 키 마스킹
        record.msg = re.sub(
            r'(api_key|token)=[\w-]+',
            r'\1=***',
            record.msg
        )
        
        # 이메일 마스킹
        record.msg = re.sub(
            r'[\w.-]+@[\w.-]+',
            lambda m: m.group(0).replace(m.group(0).split('@')[0][:-3], '***'),
            record.msg
        )
        
        return True

logger = logging.getLogger(__name__)
logger.addFilter(MaskingFilter())
```

### GDPR/규제 준수

```text
개인정보보호법:
- 개인 식별 가능한 정보는 필요한 기간만 보관
- 로그는 보통 추적 목적이므로 민감 정보 제외
- retention 정책 명확히 (규제 요구사항)

예시:
- 신용카드: 13개월 보관 (규제)
- 거래 기록: 6년 보관 (세무)
- 접근 로그: 90일 보관 (보안)
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Log Level | DEBUG, INFO, WARN, ERROR, CRITICAL |
| Structured Logging | JSON 형식의 정형화된 로그 |
| Correlation ID | 분산 시스템의 요청 추적 ID |
| Log Aggregation | 중앙 집중식 로그 수집/검색 |
| Retention | 로그 보관 기간 정책 |
| Masking | 민감 정보 숨김 처리 |

---

## Related Notes

- [Metrics](../metrics/overview.md)
- [Prometheus](../prometheus/overview.md)
