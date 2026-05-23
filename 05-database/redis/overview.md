# Redis

## 목차
- [Redis의 역할](#redis의-역할)
- [key-value model](#key-value-model)
- [data structure](#data-structure)
- [TTL](#ttl)
- [cache pattern](#cache-pattern)
- [persistence](#persistence)

## Redis의 역할
Redis는 메모리 기반 key-value 저장소다. 디스크 중심의 관계형 데이터베이스보다 빠른 응답이 필요한 곳에 자주 사용된다.

```text
application
-> Redis
-> memory
```

Redis가 빠른 이유는 주로 메모리에서 데이터를 다루고, 단순한 자료구조 명령을 매우 빠르게 처리하도록 설계되었기 때문이다.

```text
메모리 기반
간단한 명령 모델
효율적인 자료구조
단일 스레드 이벤트 루프 기반 처리 모델
```

Redis는 메인 데이터베이스를 완전히 대체하기보다 캐시, 세션 저장소, rate limit, queue, pub/sub 같은 보조 저장소로 많이 사용된다.

## key-value model
Redis의 기본 모델은 key-value다. key로 값을 저장하고 key로 값을 조회한다.

```bash
SET user:1:name shin
GET user:1:name
```

key 설계는 중요하다. 보통 namespace처럼 `:`를 사용해 의미를 나눈다.

```text
user:1:name
session:abc123
rate-limit:user:1
```

Redis key는 문자열이지만 value는 단순 문자열만 가능한 것이 아니다. list, hash, set, sorted set 같은 자료구조를 value로 사용할 수 있다.

## data structure
Redis는 다양한 자료구조를 제공한다.

```text
String       단순 값, counter
Hash         객체 필드 저장
List         순서 있는 목록, queue
Set          중복 없는 집합
Sorted Set   점수 기반 정렬 집합
Stream       append-only event log
```

예를 들어 사용자 객체 일부를 hash로 저장할 수 있다.

```bash
HSET user:1 name shin email shin@example.com
HGET user:1 name
```

카운터는 string 명령으로 처리할 수 있다.

```bash
INCR page:view:home
```

자료구조를 잘 고르면 애플리케이션에서 직접 복잡하게 처리할 일을 Redis 명령으로 단순화할 수 있다.

## TTL
TTL(Time To Live)은 key의 만료 시간이다. 일정 시간이 지나면 Redis가 key를 삭제한다.

```bash
SET session:abc123 user-1
EXPIRE session:abc123 3600
```

TTL은 캐시와 세션에서 특히 중요하다.

```text
캐시      오래된 데이터를 자동으로 제거
세션      로그인 상태 만료
rate limit 일정 시간 창이 지나면 카운터 초기화
```

TTL이 없으면 캐시 데이터가 계속 쌓여 메모리를 차지할 수 있다. Redis는 메모리 저장소이기 때문에 만료 정책과 eviction 정책을 함께 이해해야 한다.

## cache pattern
캐시는 느리거나 비싼 조회 결과를 빠르게 재사용하기 위한 저장소다.

가장 기본적인 패턴은 cache-aside다.

```text
1. 애플리케이션이 Redis에서 먼저 조회
2. cache hit이면 바로 반환
3. cache miss이면 DB 조회
4. DB 결과를 Redis에 저장
5. 결과 반환
```

```text
cache hit   캐시에 값이 있음
cache miss  캐시에 값이 없음
```

cache invalidation은 캐시를 언제 지울지 또는 갱신할지 결정하는 문제다. 이것이 어려운 이유는 원본 DB의 값이 바뀌었는데 캐시에 오래된 값이 남을 수 있기 때문이다.

```text
DB update
-> 관련 cache delete
-> 다음 조회에서 cache miss
-> 최신 DB 값으로 cache 재생성
```

캐시는 성능을 높이지만 일관성 문제를 만든다. TTL만 믿을지, 쓰기 시점에 삭제할지, event 기반으로 갱신할지 선택해야 한다.

## persistence
Redis는 메모리 기반이지만 데이터를 디스크에 저장하는 persistence 기능도 제공한다.

대표적으로 RDB snapshot과 AOF가 있다.

```text
RDB  특정 시점의 snapshot 저장
AOF  쓰기 명령 로그를 append
```

persistence를 이해해야 하는 이유는 Redis를 순수 cache로 쓸 때와 데이터 보존이 필요한 저장소로 쓸 때 운영 전략이 달라지기 때문이다.

```text
순수 cache       데이터 유실 허용 가능
세션 저장소       유실 영향 검토 필요
queue처럼 사용    유실과 중복 처리 전략 필요
```

Redis persistence는 데이터 유실 가능성을 줄여주지만, 전통적인 관계형 DB처럼 모든 상황에서 강한 durability를 제공한다고 단순히 가정하면 안 된다. 용도에 따라 RDB, AOF, replication, backup 전략을 함께 설계해야 한다.
