# Streaming

## 목차
- [Streaming의 역할](#streaming의-역할)
- [Event Stream](#event-stream)
- [Producer와 Consumer](#producer와-consumer)
- [Offset](#offset)
- [Window](#window)
- [Delivery Semantics](#delivery-semantics)
- [Late Event](#late-event)

---

## Streaming의 역할

**Streaming은 데이터가 발생하는 즉시 또는 짧은 지연으로 계속 처리하는 방식**이다. Batch와 달리 실시간성이 필요한 경우에 사용한다.

```text
Event 발생 (주문 생성)
    ↓ (밀리초 후)
Kafka Topic (메시지 저장)
    ↓
Consumer A (재고 감소)
↓
Consumer B (알림 전송)
↓
Consumer C (분석 저장)
```

### Batch vs Streaming 다시

```text
Batch: 모아서 처리
- 매일 02:00에 전날 데이터 한번에
- 처리량 높음, 지연 있음

Streaming: 즉시 처리
- 이벤트 발생하면 바로 처리
- 처리량 낮음, 지연 없음

혼합 (Lambda Architecture):
- Streaming: 실시간 (대략적)
- Batch: 정확성 (나중에 수정)
```

---

## Event Stream

### "event와 message는 어떻게 구분할 수 있는가?"

**Event는 발생한 사건 (불변), Message는 전달할 정보 (전송용)**이다.

```text
Event (불변 기록):
{
  "type": "order-created",
  "order_id": 123,
  "user_id": 456,
  "timestamp": "2026-05-23T10:15:30Z",
  "amount": 150
}
→ 발생한 그대로 저장, 변경 불가

Message (전달용):
{
  "to": "inventory-service",
  "action": "decrease_stock",
  "order_id": 123,
  "quantity": 2
}
→ 전달 목적, 변경 가능
```

### Event 특징

```text
- 시간 기반 순서
- 불변 (change log)
- 여러 consumer 가능
- 재생 가능 (replay)
```

---

## Producer와 Consumer

### Producer

```python
# Kafka Producer (이벤트 발송)
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')

# 주문 생성 이벤트
event = {
    "type": "order-created",
    "order_id": 123,
    "timestamp": "2026-05-23T10:15:30Z"
}

producer.send('orders', value=event)
```

### Consumer

```python
# Kafka Consumer (이벤트 수신)
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'orders',
    bootstrap_servers='localhost:9092',
    group_id='inventory-service',
    auto_offset_reset='earliest'
)

for message in consumer:
    event = message.value
    # 재고 감소
    decrease_inventory(event['order_id'])
```

---

## Offset

### "offset은 consumer에게 왜 중요한가?"

**Offset은 Consumer가 어디까지 읽었는지 기록해서, 실패 시 다시 읽을 수 있게 한다.**

```text
Kafka Topic (불변 로그):
[msg0][msg1][msg2][msg3][msg4][msg5]...
 0     1     2     3     4     5

Consumer A (처음):
offset=0 시작
읽은 후: offset=3
commit (저장)

Consumer A (실패 후 복구):
offset=3부터 시작 (msg4, msg5 다시 읽음)
→ 데이터 손실 없음
```

### Offset 관리

```text
자동 커밋:
- 자동으로 일정 간격 저장
- 간단하지만 손실 가능

수동 커밋:
- 처리 완료 후 명시적 커밋
- 안전하지만 복잡
```

---

## Window

### "window 집계는 왜 필요한가?"

**Unbounded 스트림에서는 집계가 끝나지 않으므로 시간/크기 기반 window로 끊어서 처리**해야 한다.

```text
실시간 요청 스트림:
[req1][req2][req3][req4][req5][req6]...

집계 불가능 (무한히 계속)

시간 window (5분 단위):
Window 1 (00:00-00:05): [req1, req2, req3] → 평균 응답시간
Window 2 (00:05-00:10): [req4, req5, req6] → 평균 응답시간
```

### Window 종류

```text
Tumbling Window (비겹침):
|--5분--|--5분--|--5분--|
모든 시간 포함, 겹침 없음

Sliding Window (겹침):
|--5분--|
  |--5분--|
    |--5분--|
중복 처리, 최신 분석

Session Window (활동 기반):
|활동 1분 그룹|  휴지  |활동 2분 그룹|
사용자 활동 묶음
```

---

## Delivery Semantics

### "exactly-once는 왜 어렵고 비싼가?"

**정확히 한 번 전달 (exactly-once)을 보장하려면 consumer 상태 저장과 트랜잭션이 필수**라 복잡하고 비싸다.**

```text
At-most-once (최대 한번):
- 빠름, 저렴
- 데이터 손실 가능
- 예: 로그 수집

At-least-once (최소 한번):
- 중간
- 중복 가능
- Idempotent 처리 필수
- 예: 주문 처리

Exactly-once (정확히 한번):
- 느림, 비쌈
- 데이터 손실/중복 없음
- 분산 트랜잭션 필요
- 예: 금융 거래
```

### Exactly-once 구현

```text
어려운 이유:
1. Consumer 상태 저장
   - offset 저장은 언제? 처리 전? 후?
   - 처리 중 실패 시?

2. 원자성
   - 데이터 저장 + offset 저장
   - 둘 다 성공해야 함

3. 분산 시스템
   - 여러 service 간 조정
   - 네트워크 실패 처리
```

---

## Late Event

### 늦은 이벤트 처리

```text
예: 모바일 앱 오프라인
1. 13:00 주문 (이벤트 생성)
2. 네트워크 없음
3. 14:30 네트워크 복구
4. 이벤트 전송
→ 13:00 window에 14:30에 도착

문제: 13:00-13:05 window는 이미 닫혔음

해결책:
✓ Late window (grace period)
  - 13:10까지 늦은 이벤트 허용
  
✓ 재계산
  - 13:00-13:05 결과 다시 계산
  
✓ 별도 처리
  - 늦은 이벤트는 따로 기록
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Event Stream | 시간 순서 불변 기록 |
| Producer | 이벤트 발송 |
| Consumer | 이벤트 수신 처리 |
| Offset | Consumer의 읽은 위치 |
| Window | 시간/크기 기반 집계 |
| Delivery | At-most/At-least/Exactly-once |

---

## Related Notes

- [Data Pipeline](../data-pipeline/overview.md)
- [Batch Processing](../batch-processing/overview.md)
