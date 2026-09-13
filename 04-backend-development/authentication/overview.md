# Authentication

## 목차
- [인증과 인가](#인증과-인가)
- [session](#session)
- [cookie](#cookie)
- [JWT](#jwt)
- [OAuth 2.0](#oauth-20)
- [password hashing](#password-hashing)

## 인증과 인가
인증(authentication)은 사용자가 누구인지 확인하는 과정이다. 인가(authorization)는 인증된 사용자가 특정 자원이나 행동에 접근할 수 있는지 판단하는 과정이다.

```text
인증  당신은 누구인가?
인가  당신은 이것을 할 권한이 있는가?
```

두 개념은 반드시 구분해야 한다. 로그인에 성공했다고 모든 자원에 접근할 수 있는 것은 아니다.

```text
로그인 성공
-> 사용자 ID 확인
-> 관리자 페이지 접근 요청
-> 관리자 권한이 있는지 검사
```

예를 들어 일반 사용자는 자신의 주문 목록은 볼 수 있지만, 다른 사용자의 주문이나 관리자 통계는 볼 수 없어야 한다.

## session
session은 서버가 로그인 상태를 저장하는 방식이다. 사용자가 로그인하면 서버는 session을 만들고, 클라이언트에는 session id를 전달한다.

```text
login
-> server creates session
-> session id 전달
-> 이후 요청에서 session id로 사용자 식별
```

브라우저에서는 보통 session id를 cookie에 담는다.

```text
Set-Cookie: SESSION=abc123
```

이후 브라우저는 같은 사이트 요청에 cookie를 자동으로 포함한다.

```text
Cookie: SESSION=abc123
```

session 방식은 서버가 상태를 저장하므로 강제 로그아웃, 세션 만료, 서버 측 권한 변경 반영이 쉽다. 대신 서버가 session storage를 관리해야 한다.

## cookie
cookie는 브라우저가 저장하고 요청에 자동으로 포함할 수 있는 작은 데이터다. 인증에서는 session id나 refresh token을 담는 데 자주 쓰인다.

```http
Set-Cookie: SESSION=abc123; HttpOnly; Secure; SameSite=Lax
```

중요한 cookie 속성은 다음과 같다.

```text
HttpOnly  JavaScript에서 cookie 접근을 막아 XSS 피해를 줄임
Secure    HTTPS에서만 전송
SameSite  cross-site 요청에 cookie를 보낼지 제어
Max-Age   만료 시간 설정
```

cookie는 자동으로 요청에 포함되기 때문에 편리하지만 CSRF 같은 공격을 고려해야 한다. API 인증 설계에서는 cookie 속성과 CSRF 방어를 함께 생각해야 한다.

## JWT
JWT(JSON Web Token)는 JSON 형태의 claim을 서명한 token이다. 서버가 발급하고 클라이언트가 이후 요청에 포함한다.

```text
header.payload.signature
```

요청 예시는 다음과 같다.

```http
Authorization: Bearer eyJhbGciOi...
```

JWT의 장점은 token 자체에 사용자 식별자나 권한 같은 claim을 담을 수 있고, 서버가 서명을 검증해 위조 여부를 판단할 수 있다는 점이다.

```json
{
  "sub": "user-1",
  "role": "USER",
  "exp": 1710000000
}
```

session과 JWT의 핵심 차이는 상태 저장 위치다.

```text
session  서버가 로그인 상태를 저장
JWT      token에 claim을 담고 서버는 서명 검증
```

JWT는 stateless하게 쓰기 쉽지만, 발급된 token을 즉시 무효화하기 어렵다. 그래서 만료 시간, refresh token, blacklist 전략을 함께 고려해야 한다.

## OAuth 2.0
OAuth 2.0은 본질적으로 로그인 기술이라기보다 **권한 위임 프로토콜**이다. 사용자가 어떤 애플리케이션에게 자신의 특정 자원 접근 권한을 위임할 수 있게 한다.

```text
사용자
-> authorization server
-> client application에 access token 발급
-> resource server 접근
```

예를 들어 어떤 앱이 사용자의 Google Calendar를 읽고 싶을 때, 사용자는 Google 로그인 화면에서 권한을 허용하고 앱은 access token을 받는다.

```text
client app
-> Google authorization server
-> access token
-> Google Calendar API
```

"소셜 로그인"은 OAuth 2.0과 OpenID Connect를 함께 사용해 로그인처럼 제공되는 경우가 많다. 정확히는 OAuth는 권한 위임, OpenID Connect는 인증 정보를 표준화해 제공하는 계층이다.

```text
OAuth 2.0        권한 위임
OpenID Connect   사용자 인증 정보 제공
```

## password hashing
비밀번호는 절대 평문으로 저장하면 안 된다. 데이터베이스가 유출되면 모든 사용자의 비밀번호가 그대로 노출되기 때문이다.

비밀번호는 단방향 hash로 저장해야 한다. 또한 salt를 사용해 같은 비밀번호도 서로 다른 hash가 되게 해야 한다.

```text
password
-> salt 추가
-> slow hash
-> hash 저장
```

일반적인 빠른 hash 함수만 쓰면 공격자가 대량으로 추측을 시도하기 쉽다. 비밀번호 저장에는 bcrypt, scrypt, Argon2처럼 의도적으로 느린 password hashing 알고리즘을 사용한다.

```text
저장하면 안 됨   plain text password
권장 방식        salted password hash
```

로그인 시에는 사용자가 입력한 비밀번호에 같은 방식의 hash 검증을 수행하고, 저장된 hash와 비교한다.

```text
입력 password
-> hash 검증
-> 저장된 hash와 비교
-> 성공 또는 실패
```
