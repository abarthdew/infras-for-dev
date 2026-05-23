# Redis

## 목차
- Redis의 역할
- key-value model
- data structure
- TTL
- cache pattern
- persistence

## 기초 개념
Redis는 메모리 기반 key-value 저장소다. 캐시, 세션 저장소, rate limit, queue, pub/sub 등에 자주 사용된다.

```text
app -> Redis -> memory
```

## 간단한 예시
```bash
SET user:1:name shin
GET user:1:name
EXPIRE user:1:name 60
```

## 반드시 알아야 할 질문
- Redis는 왜 빠른가?
- cache miss와 cache invalidation은 무엇인가?
- TTL은 어떤 문제를 해결하는가?
- Redis persistence는 왜 선택적으로 이해해야 하는가?
