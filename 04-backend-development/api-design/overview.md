# API Design

## 목차
- Resource와 endpoint
- HTTP method
- status code
- request/response schema
- pagination
- versioning

## 기초 개념
API 설계는 클라이언트와 서버가 어떤 계약으로 데이터를 주고받을지 정하는 일이다. 좋은 API는 예측 가능하고, 상태 코드와 응답 형식이 일관적이다.

```text
client -> HTTP request -> API server -> HTTP response
```

## 간단한 예시
```http
GET /users/1 HTTP/1.1
Accept: application/json
```

```json
{
  "id": 1,
  "name": "shin"
}
```

## 반드시 알아야 할 질문
- resource 중심 설계는 무엇인가?
- `POST`, `PUT`, `PATCH`는 무엇이 다른가?
- 에러 응답은 어떤 형식으로 통일해야 하는가?
- pagination은 offset과 cursor 중 언제 무엇을 쓰는가?
