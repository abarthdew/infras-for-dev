# API Design

## 목차
- [Resource와 endpoint](#resource와-endpoint)
- [HTTP method](#http-method)
- [status code](#status-code)
- [request/response schema](#requestresponse-schema)
- [pagination](#pagination)
- [versioning](#versioning)

## Resource와 endpoint
API 설계에서 resource는 클라이언트가 다루는 대상이다. 사용자, 주문, 상품, 댓글 같은 개념이 resource가 될 수 있다. endpoint는 그 resource에 접근하는 URL 경로다.

```text
resource  User
endpoint  /users
```

resource 중심 설계는 동작 이름보다 대상을 중심으로 URL을 잡는 방식이다.

```http
GET /users/1
POST /users
GET /orders?userId=1
```

다음처럼 동사를 URL에 과도하게 넣는 방식은 일관성이 약해질 수 있다.

```http
POST /createUser
POST /deleteUser
```

물론 모든 API가 완벽히 REST 형태일 필요는 없다. 중요한 것은 클라이언트가 예측할 수 있는 규칙을 유지하는 것이다.

## HTTP method
HTTP method는 resource에 어떤 행동을 할지 나타낸다.

```text
GET     조회
POST    생성 또는 명령성 작업
PUT     전체 교체
PATCH   부분 수정
DELETE  삭제
```

`POST`, `PUT`, `PATCH`는 자주 헷갈린다.

```text
POST   새 resource 생성 또는 처리 요청
PUT    resource 전체를 주어진 표현으로 교체
PATCH  resource 일부 필드만 수정
```

예시는 다음과 같다.

```http
POST /users
```

```json
{
  "name": "shin"
}
```

```http
PATCH /users/1
```

```json
{
  "name": "kim"
}
```

method 선택은 캐싱, 재시도, idempotency와도 연결된다. 예를 들어 `GET`은 서버 상태를 바꾸지 않아야 한다.

## status code
HTTP status code는 요청 처리 결과를 숫자로 표현한다.

```text
2xx  성공
3xx  리다이렉션
4xx  클라이언트 오류
5xx  서버 오류
```

자주 쓰는 코드는 다음과 같다.

```text
200 OK                  요청 성공
201 Created             생성 성공
204 No Content          성공했지만 응답 본문 없음
400 Bad Request         요청 형식 오류
401 Unauthorized        인증 필요 또는 실패
403 Forbidden           권한 없음
404 Not Found           resource 없음
409 Conflict            현재 상태와 충돌
500 Internal Server Error 서버 내부 오류
```

상태 코드는 클라이언트가 다음 행동을 결정하는 데 중요하다. 예를 들어 `401`이면 로그인 또는 토큰 갱신이 필요하고, `403`이면 로그인해도 권한이 없다는 의미에 가깝다.

## request/response schema
schema는 요청과 응답의 데이터 구조 계약이다. 좋은 API는 필드 이름, 타입, 필수 여부, 에러 형식을 일관되게 유지한다.

```json
{
  "id": 1,
  "name": "shin",
  "email": "shin@example.com"
}
```

에러 응답도 통일하는 것이 좋다.

```json
{
  "code": "USER_NOT_FOUND",
  "message": "User not found",
  "requestId": "abc-123"
}
```

에러 응답 형식이 매번 다르면 클라이언트가 예외 처리를 안정적으로 하기 어렵다.

```text
좋은 schema
-> 예측 가능
-> 문서화 가능
-> 테스트 가능
-> 클라이언트 구현 단순화
```

## pagination
pagination은 많은 목록 데이터를 나누어 가져오는 방식이다. 대표적으로 offset pagination과 cursor pagination이 있다.

offset 방식은 페이지 번호나 시작 위치를 기준으로 한다.

```http
GET /users?offset=20&limit=10
```

장점은 단순하다는 것이다. 단점은 데이터가 계속 추가/삭제되는 상황에서 중복이나 누락이 생길 수 있고, offset이 커지면 성능이 나빠질 수 있다는 점이다.

cursor 방식은 마지막으로 본 항목의 기준값을 다음 요청에 전달한다.

```http
GET /users?cursor=eyJpZCI6MjAxfQ&limit=10
```

cursor 방식은 무한 스크롤, 실시간으로 변하는 목록에 더 적합한 경우가 많다.

```text
offset  단순한 관리자 목록, 작은 데이터
cursor  큰 데이터, 무한 스크롤, 자주 변하는 목록
```

## versioning
versioning은 API 계약을 변경할 때 기존 클라이언트를 보호하기 위한 전략이다.

```http
GET /v1/users/1
GET /v2/users/1
```

API는 한번 공개되면 여러 클라이언트가 의존한다. 필드를 갑자기 삭제하거나 의미를 바꾸면 기존 앱이 깨질 수 있다.

변경은 크게 두 종류로 나눌 수 있다.

```text
호환 변경     새 optional 필드 추가
비호환 변경   필드 삭제, 타입 변경, 의미 변경
```

versioning은 비호환 변경을 안전하게 도입하기 위한 방법이다. 다만 버전이 많아지면 서버 유지 비용이 증가한다.

좋은 API 변경 전략은 다음과 같다.

```text
기존 응답 필드의 의미를 함부로 바꾸지 않기
새 필드는 가능하면 optional로 추가하기
비호환 변경은 새 version으로 제공하기
deprecated 기간을 명확히 두기
```
