# Grafana

## 목차
- [Grafana의 역할](#grafana의-역할)
- [Data Source](#data-source)
- [Dashboard](#dashboard)
- [Panel](#panel)
- [Variable](#variable)
- [Alert](#alert)
- [베스트 프랙티스](#베스트-프랙티스)

---

## Grafana의 역할

**Grafana는 Prometheus, Elasticsearch, PostgreSQL 같은 데이터 소스를 시각화하는 대시보드 도구**다. 복잡한 metrics를 그래프, 게이지, 테이블로 표현해서 직관적으로 이해할 수 있게 한다.

```text
Data Source (Prometheus, Elasticsearch, 등)
    ↓
Query (PromQL, SQL, 등)
    ↓
Panel (Graph, Gauge, Table, 등)
    ↓
Dashboard (여러 Panel 조합)
    ↓
Alert (임계값 기반)
    ↓
사용자 (직관적 이해)
```

### Grafana vs Prometheus

```text
Prometheus:
- metrics 수집 및 저장
- 쿼리 가능 (PromQL)
- 기본적인 그래프 표시
- 알림 설정 가능

Grafana:
- 다양한 시각화 (고급 차트)
- 여러 데이터 소스 통합
- 대시보드 구성 및 공유
- 사용자 친화적 UI
- 권한 관리
```

---

## Data Source

### Data Source 추가

```text
지원하는 데이터 소스:
├─ Prometheus (시계열 metrics)
├─ Elasticsearch (로그, 이벤트)
├─ PostgreSQL, MySQL (관계형 DB)
├─ InfluxDB (시계열 DB)
├─ CloudWatch (AWS metrics)
├─ DataDog (외부 모니터링)
└─ Loki (로그 집계)
```

### Grafana UI에서 설정

```text
1. 왼쪽 메뉴 → Configuration → Data Sources
2. "Add data source" 클릭
3. Prometheus 선택
4. URL: http://prometheus:9090
5. "Save & test" → "Data source is working"
6. 완료
```

### 데이터 소스 쿼리

```
Prometheus 데이터 소스를 추가하면,
dashboard에서 PromQL 쿼리 가능:

rate(http_requests_total[5m])
histogram_quantile(0.95, http_request_duration_seconds)
```

---

## Dashboard

### "좋은 dashboard는 어떤 질문에 답해야 하는가?"

**좋은 Dashboard는 사용자의 구체적 질문에 빠르게 답할 수 있도록 구성**되어야 한다.

```text
나쁜 Dashboard:
- 모든 metrics를 다 넣음
- 사용자가 원하는 정보를 찾기 어려움
- 로딩 시간이 김
- 스크롤이 무한정

좋은 Dashboard:
- 한눈에 상황을 파악 (RED)
- 구체적 질문에 답함
- 빠른 로딩 (< 3초)
- 적절한 크기 (한 화면)
```

### Dashboard 설계 원칙

```text
SRE Dashboard (서비스 건강성):
- Request Rate (RPS)
- Error Rate (%)
- p99 Latency
- Availability (%)

Database Dashboard:
- 쿼리 성능 (평균, p99)
- Slow Query 수
- Connection 수
- Lock 대기 시간

Infrastructure Dashboard:
- CPU Usage
- Memory Usage
- Disk Usage
- Network I/O
```

### Dashboard 생성

```text
1. Grafana 홈 → "Create Dashboard"
2. "Add Panel" 클릭
3. Data source: Prometheus 선택
4. Query: rate(http_requests_total[5m])
5. Panel title: "Request Rate"
6. Save
```

---

## Panel

### "panel은 metrics를 어떻게 오해하게 만들 수 있는가?"

**Panel의 설정 (범위, 임계값, 색상)이 부정확하면 metrics를 잘못 해석하게 한다.**

```text
나쁜 Panel 설정:

[Graph] y축 범위: 0-100
CPU: 45% → 거의 절반처럼 보임 (실제로 충분)

[Gauge] 빨강: > 80, 녹색: < 40
현재 값: 45% → 녹색 (안전) ✓
올바른 판단

문제 예시:

[Graph] y축 범위: 45-55
CPU: 45% → 맨 아래 (위험해 보임)
CPU: 48% → 상승 추세 (실제로는 정상)
→ 오해 유발!
```

### Panel 종류

```text
Graph (선 그래프)
- 시간대 추이 표시
- 여러 메트릭 비교
- 예: CPU over time, RPS trend

Gauge (게이지)
- 현재값 한눈에 표시
- 임계값 시각화 (색상)
- 예: 현재 메모리 사용률

Table (테이블)
- 상세 데이터
- TOP N 표시
- 예: 느린 쿼리 Top 10

Stat (통계)
- 한 숫자 강조
- 비교 (지난 기간 vs 현재)
- 예: "일일 매출 $50,000 (+10%)"

Heatmap (히트맵)
- 2D 분포 시각화
- 시간과 값의 밀도
- 예: 응답 시간 히트맵
```

### Panel 최적 설정

```text
✓ 올바른 y축 범위 (데이터 전체 범위 표시)
✓ 적절한 임계값 (빨강, 노랑, 초록)
✓ 명확한 제목 (무엇을 보는가?)
✓ 단위 표시 (%, ms, count)
✓ Legend (여러 시리즈 구분)
✗ 과도한 panel (한 dashboard에 20개 이상)
```

---

## Variable

### "variable은 언제 유용한가?"

**Variable은 하나의 Dashboard에서 필터/파라미터를 사용해 다양한 환경/서버를 동적으로 선택할 수 있게 한다.**

```text
변수 없이:
- 서버 A 대시보드
- 서버 B 대시보드
- 서버 C 대시보드
→ 같은 대시보드를 여러 개 생성

변수 사용:
- 하나의 대시보드
- "Instance" 드롭다운
  - server-1
  - server-2
  - server-3
- 선택하면 자동으로 metrics 변경
```

### Variable 생성

```text
1. Dashboard 편집 → Settings (우상단 gear)
2. Variables 탭
3. "Add variable" 클릭
4. Name: instance
5. Type: Query
6. Data source: Prometheus
7. Query: label_values(up, instance)
8. Save
```

### Variable 사용

```
Query에서:
rate(http_requests_total{instance="$instance"}[5m])

$instance가 "server-1"이면:
rate(http_requests_total{instance="server-1"}[5m])

$instance가 "server-2"이면:
rate(http_requests_total{instance="server-2"}[5m])
```

### Variable 타입

```text
Query: Prometheus 쿼리로 값 목록 생성
Const: 고정 값 (드롭다운 선택 안 함)
Text box: 사용자가 직접 입력
Interval: 시간 범위 (5m, 1h, 1d)
Custom: 수동으로 목록 작성
```

---

## Alert

### "alert과 dashboard는 역할이 어떻게 다른가?"

**Alert는 자동 감시 (주기적 체크), Dashboard는 수동 관찰 (필요할 때 봄)**이다.

```text
Alert (능동적):
- 문제 발생 → 자동 알림 (Slack, Email)
- 24/7 모니터링
- 대응 필요성 전파

Dashboard (수동적):
- "지금 상황을 봐야 하나?" → 직접 확인
- 필요할 때만 봄
- 상세 분석용
```

### Grafana Alert 설정

```text
1. Panel 편집
2. Alert 탭
3. "Create alert rule"
4. 조건 설정: CPU > 80% for 5m
5. Notification: Slack 채널 지정
6. Save
```

### Alert vs Dashboard 사용 사례

```text
Alert를 사용:
- 서비스 다운
- 에러율 급증
- 디스크 부족 (곧 영향)
- 리소스 고갈 (곧 성능 저하)

Dashboard를 사용:
- "최근 며칠 추세가 어때?"
- "배포 전후 성능 비교"
- "특정 시간대 원인 분석"
- "용량 계획 (향후 3개월)"
```

---

## 베스트 프랙티스

### Dashboard 설계

```text
1. 역할별 Dashboard
   ├─ SRE (서비스 건강성)
   ├─ DBA (데이터베이스)
   ├─ DevOps (인프라)
   └─ 개발자 (자신 서비스)

2. 계층적 구성
   ├─ 높은 수준 (1개 화면)
   │  └─ 이상 감지
   └─ 상세 수준 (여러 화면)
      └─ 원인 분석

3. 성능 최적화
   ├─ 필요한 metrics만 표시
   ├─ 적절한 시간 범위 (1h, 24h)
   └─ Panel 개수 제한 (< 12개)
```

### Variable 활용

```
좋은 사례:
- Instance/Server 필터
- Environment (prod, staging, dev)
- Service (api, db, cache)
- Time range

나쁜 사례:
- 모든 label을 variable로 만듦
- 사용되지 않는 variable
```

### 공유 및 권한

```text
대시보드 공유:
1. 우상단 Share 버튼
2. "Public dashboard" (누구나 접근)
3. "Link" (특정 URL로만 접근)
4. "Snapshot" (현재 상태 저장)
   → 다른 사람도 과거 데이터 볼 수 있음

권한:
- Admin: 생성, 수정, 삭제, 공유
- Edit: 수정만
- View: 읽기만
```

---

## 실전 예시: SRE Dashboard

```
대시보드 구성:

[Row 1: 개요]
├─ Green (모든 정상) / Red (문제 있음)
├─ Availability (%)
├─ Requests/sec
└─ Error Rate (%)

[Row 2: 성능]
├─ p50 Latency
├─ p95 Latency
├─ p99 Latency
└─ Max Latency

[Row 3: 에러 분석]
├─ Error by Status Code
├─ Error by Endpoint
└─ Error Trend

[Row 4: 리소스]
├─ CPU Usage
├─ Memory Usage
└─ Network I/O

[Row 5: 상세 분석]
└─ 느린 Endpoint Top 10

선택 가능 Variable:
- Instance: server-1, server-2, ...
- Environment: prod, staging
- Time: 1h, 24h, 7d
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Data Source | Prometheus, Elasticsearch 등 데이터 연결 |
| Dashboard | 여러 Panel의 조합 |
| Panel | 그래프, 게이지 등 시각화 요소 |
| Variable | 필터/파라미터 (동적 선택) |
| Alert | 임계값 기반 자동 알림 |

---

## Related Notes

- [Prometheus](../prometheus/overview.md)
- [Logging](../logging/overview.md)
