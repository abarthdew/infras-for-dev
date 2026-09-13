# Prometheus

## 목차
- [Prometheus의 역할](#prometheus의-역할)
- [Metrics 수집](#metrics-수집)
- [Scrape Model](#scrape-model)
- [Exporter](#exporter)
- [PromQL](#promql)
- [Alert Rule](#alert-rule)
- [저장 및 유지](#저장-및-유지)

---

## Prometheus의 역할

**Prometheus는 시스템과 애플리케이션의 metrics를 주기적으로 수집하고, 저장하고, 질의하는 오픈소스 모니터링 도구**다. Pull 방식으로 대상 endpoint를 주기적으로 스크래핑한다.

```text
Application (metrics 내보냄)
    ↓ /metrics endpoint
Prometheus (주기적 수집, 저장)
    ↓
TSDB (시계열 데이터베이스)
    ↓
PromQL (질의)
    ├─ Grafana (시각화)
    ├─ AlertManager (알림)
    └─ API (외부 시스템)
```

### Push vs Pull

```text
Push 방식 (InfluxDB, Graphite):
애플리케이션 → "metrics를 보냄" → 모니터링 시스템
장점: 실시간
단점: 애플리케이션이 수신 보장해야 함

Pull 방식 (Prometheus):
모니터링 시스템 → "metrics를 가져가" → /metrics endpoint
장점: 중앙에서 제어, 간단
단점: 약간의 지연
```

---

## Metrics 수집

### "counter, gauge, histogram은 무엇이 다른가?"

**Counter는 증가만 하고, Gauge는 증감이 자유롭고, Histogram은 분포를 기록**한다.

```text
Counter (카운터)
├─ 계속 증가하는 값
├─ 예: http_requests_total, errors_total
├─ 특징: 절대 감소, 리셋되면 0부터 시작
└─ 연산: rate() 함수로 초당 변화량 계산

Gauge (게이지)
├─ 현재값 (증감 자유)
├─ 예: memory_usage_bytes, cpu_usage_percent
├─ 특징: 순간값, 자유롭게 변함
└─ 연산: 직접 값 사용 가능

Histogram (히스토그램)
├─ 값의 분포 (버킷)
├─ 예: http_request_duration_seconds
├─ 내부: _bucket (버킷별 개수), _sum (합계), _count (개수)
└─ 연산: 퍼센타일 (p50, p95, p99) 계산
```

### 수집 설정

```yaml
# prometheus.yml
global:
  scrape_interval: 15s      # 15초마다 수집
  scrape_timeout: 10s       # 수집 시도 최대 10초
  evaluation_interval: 15s  # alert 평가 간격

scrape_configs:
  - job_name: 'api-server'
    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
    metrics_path: '/metrics'
    scrape_interval: 5s      # 이 job은 5초마다

  - job_name: 'database'
    static_configs:
      - targets: ['localhost:5432']
    scrape_interval: 30s     # 이 job은 30초마다
```

---

## Scrape Model

### "scrape interval은 어떤 영향을 주는가?"

**Scrape interval은 Prometheus가 metrics를 얼마나 자주 수집하는지 결정하며, 저장 공간과 응답 시간에 영향**을 미친다.

```text
Scrape interval: 5초
├─ 시간당: 12 requests/분 * 60분 = 720 data points
├─ 일일: 720 * 24 = 17,280 data points
├─ 연간: 17,280 * 365 = 6.3M data points
└─ 저장 공간: 매우 높음, 비용 증가

Scrape interval: 15초 (기본)
├─ 시간당: 240 data points
├─ 일일: 5,760 data points
├─ 연간: 2.1M data points
└─ 저장 공간: 중간

Scrape interval: 60초
├─ 시간당: 60 data points
├─ 일일: 1,440 data points
├─ 연간: 525K data points
└─ 저장 공간: 낮음, 하지만 세밀함 감소
```

### Interval별 트레이드오프

```text
짧은 interval (5초):
장점: 매우 세밀한 모니터링, 빠른 감지
단점: 높은 저장 비용, 높은 CPU 사용, 네트워크 부하

중간 interval (15초, 기본):
장점: 좋은 균형
단점: 없음 (대부분의 경우 충분)

긴 interval (60초+):
장점: 낮은 비용, 낮은 부하
단점: 느린 감지, 세부 정보 손실
```

### Scrape 실패 처리

```text
목표 서버가 응답 안 함:
1. Prometheus가 /metrics 요청
2. Timeout (10초) 내 응답 없음
3. "up" 메트릭 = 0
4. 이전 데이터는 보존 (약간의 지연으로 보여짐)

복구:
- 서버 재시작 후 자동 복구
- "up" 메트릭 = 1
- alert 자동 해제
```

---

## Exporter

### Exporter란?

**Exporter는 Prometheus 형식으로 metrics를 제공하는 애플리케이션**이다. 대상 시스템이 Prometheus를 지원하지 않으면 Exporter로 변환한다.

```text
시스템/애플리케이션
    ↓ (자체 형식의 metrics 제공)
Exporter (변환기)
    ↓ (Prometheus 형식으로 변환)
/metrics endpoint
    ↓
Prometheus (수집)
```

### 주요 Exporter

```text
Node Exporter
├─ 호스트 시스템 메트릭 (CPU, Memory, Disk, Network)
├─ URL: localhost:9100/metrics
└─ 설치: Linux, macOS, Windows

MySQL Exporter
├─ MySQL 데이터베이스 메트릭 (쿼리, 연결, 테이블)
├─ URL: localhost:9104/metrics
└─ 설치: MySQL 서버와 분리

Nginx Exporter
├─ Nginx 메트릭 (요청, 연결, 버퍼)
├─ URL: localhost:9113/metrics
└─ 설치: Nginx와 분리

Custom Exporter
├─ 자신의 애플리케이션 메트릭을 노출
├─ 언어별 클라이언트 라이브러리 사용
└─ 예: Python prometheus_client
```

### Exporter 설치

```bash
# Node Exporter 설치 (Linux)
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.6.1.linux-amd64.tar.gz
cd node_exporter-1.6.1.linux-amd64
./node_exporter &

# Prometheus 설정
# prometheus.yml에 추가
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

# 확인
curl http://localhost:9100/metrics
```

---

## PromQL

### "PromQL의 `rate`는 왜 필요한가?"

**`rate()` 함수는 Counter의 증가량을 초당 변화율로 계산해서 초당 처리량을 알 수 있게 해준다.**

```text
문제:
http_requests_total = 1000000 (시작 후 누적)
http_requests_total = 1000100 (100개 처리)
→ 차이는 100개지만 얼마나 빠른가? (초당? 분당?)

해결:
rate(http_requests_total[5m])
→ 지난 5분간 초당 평균 변화량 = 0.333 요청/초
(100개 증가 / 300초)
```

### PromQL 기본 쿼리

```promql
# 현재값 직접 조회
memory_usage_bytes

# 초당 변화율 (Counter)
rate(http_requests_total[5m])
# 지난 5분간 초당 평균 증가량

# 순간 변화율 (빠른 변화 감지)
irate(http_requests_total[1m])
# 지난 1분간 마지막 2개 샘플로 계산 (급격한 변화 감지)

# 합계
http_requests_total{job="api"}

# 필터
http_requests_total{status="200"}  # status 200만
http_requests_total{job=~"api.*"}  # job이 "api"로 시작
```

### PromQL 집계

```promql
# 합계
sum(http_requests_total)
sum(http_requests_total) by (status)

# 평균
avg(memory_usage_percent)
avg(memory_usage_percent) by (instance)

# 최대/최소
max(http_request_duration_seconds)
min(cpu_usage_percent)

# 백분위수 (Histogram)
histogram_quantile(0.95, http_request_duration_seconds)
# p95 (상위 5%보다 빠름)

# TOP N
topk(5, http_requests_total)
# 가장 많은 요청 5개 endpoint
```

### PromQL 실무 예시

```promql
# 실시간 RPS (Request Per Second)
rate(http_requests_total[1m])

# 에러율 (%)
100 * rate(http_requests_total{status=~"5.."}[5m]) / 
rate(http_requests_total[5m])

# p99 응답 시간
histogram_quantile(0.99, http_request_duration_seconds)

# 메모리 사용률이 80% 초과인 인스턴스
memory_usage_percent{instance=~".+"} > 80

# CPU 사용률 급증 (1분 vs 5분)
irate(cpu_usage_percent[1m]) > 1.1 * rate(cpu_usage_percent[5m])
```

---

## Alert Rule

### "alert은 어느 조건에서 울려야 하는가?"

**Alert은 문제를 사용자가 발견하기 전에 자동으로 감지하는 규칙**이다. 심각도와 응답 시간을 고려해 설정해야 한다.

```text
Alert 레벨:

Critical (긴급)
├─ 서비스 중단 또는 거의 중단 상태
├─ 응답 시간: 즉시
├─ 예: 가용성 < 99%, 에러율 > 5%, DB 다운
└─ 즉각적 조치 필요

Warning (경고)
├─ 곧 문제가 될 가능성
├─ 응답 시간: 1시간 내
├─ 예: 메모리 > 80%, 디스크 > 70%, 느린 쿼리
└─ 주의 깊게 모니터링

Info (정보)
├─ 참고만 필요
├─ 응답 시간: 당일
├─ 예: 배포 완료, 설정 변경
└─ 로그에만 기록
```

### Alert Rule 설정

```yaml
groups:
  - name: app_alerts
    interval: 30s
    rules:
      # 1. Error Rate 높음
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 2m  # 2분 이상 지속되면 alert
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} (>5%)"

      # 2. 느린 응답 시간
      - alert: SlowResponseTime
        expr: histogram_quantile(0.99, http_request_duration_seconds) > 2
        for: 5m
        annotations:
          summary: "Slow response time"
          description: "p99 latency is {{ $value }}s"

      # 3. 높은 메모리 사용
      - alert: HighMemoryUsage
        expr: memory_usage_percent > 80
        for: 5m
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"

      # 4. 서비스 다운
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        annotations:
          summary: "{{ $labels.job }} is down"
          description: "{{ $labels.instance }} has been down for >1m"
```

### Alert 조건 설정 팁

```text
False Positive 줄이기:
✓ "for" 파라미터 사용 (순간 스파이크 무시)
✓ 적절한 threshhold (너무 낮으면 오탐)
✓ 논리 결합 (여러 조건 동시 만족)

False Negative 줄이기:
✓ 여러 각도에서 모니터링
✓ 부분 장애도 감지
✓ 정기적 alert rule 검토

예:
❌ 나쁜: alert if cpu > 90% for 10s (오탐)
✓ 좋은: alert if cpu > 90% for 2m (안정적)
✓ 좋은: alert if cpu > 80% for 5m (미리 감지)
```

---

## 저장 및 유지

### Retention

```yaml
# prometheus.yml
global:
  retention: 15d  # 15일 보관
  # 또는
  retention_size: 512GB  # 디스크 크기 기준

# 수동으로 설정
--storage.tsdb.retention.time=30d
--storage.tsdb.retention.size=500GB
```

### 백업

```bash
# Prometheus snapshot 생성 (백업)
curl -X POST http://localhost:9090/api/v1/admin/tsdb/snapshot

# 출력: {"data":{"name":"20260523T101530Z-6a9fea1524ba6ee6"}}

# 백업 디렉토리
/prometheus/snapshots/20260523T101530Z-6a9fea1524ba6ee6/

# 복구
rm -rf /prometheus/wal/*
cp -r /backup/snapshots/20260523T101530Z-6a9fea1524ba6ee6/ /prometheus/wal/
```

---

## 실전 Prometheus 설정

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
  external_labels:
    monitor: 'prod-monitor'

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - localhost:9093  # AlertManager

rule_files:
  - "/etc/prometheus/rules/*.yml"

scrape_configs:
  # API 서버
  - job_name: 'api-server'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:8080']

  # 데이터베이스
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']

  # 시스템 메트릭 (Node Exporter)
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

  # Prometheus 자신
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # 서비스 디스커버리 (Consul)
  - job_name: 'consul'
    consul_sd_configs:
      - server: 'localhost:8500'
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Pull Model | Prometheus가 주기적으로 metrics 수집 |
| Scrape | metrics endpoint에서 데이터 가져오기 |
| Exporter | 대상 시스템 metrics을 Prometheus 형식으로 변환 |
| PromQL | Prometheus 쿼리 언어 |
| rate() | Counter의 초당 변화율 계산 |
| Alert | 조건 기반 자동 알림 |

---

## Related Notes

- [Metrics](../metrics/overview.md)
- [Grafana](../grafana/overview.md)
