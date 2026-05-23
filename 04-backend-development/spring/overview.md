# Spring

## 목차
- Spring Framework와 Spring Boot
- IoC와 DI
- Bean과 ApplicationContext
- Web MVC
- Data access
- Transaction

## 기초 개념
Spring은 Java 애플리케이션을 구성하고 실행하는 프레임워크다. 핵심은 객체 생성과 의존성 연결을 프레임워크가 관리한다는 점이다.

```text
Controller -> Service -> Repository
```

## 간단한 예시
```java
@RestController
class HelloController {
  @GetMapping("/hello")
  String hello() {
    return "hello";
  }
}
```

## 반드시 알아야 할 질문
- DI는 왜 테스트와 구조화에 유리한가?
- Bean lifecycle은 왜 중요한가?
- `@Transactional`은 어디에 붙여야 하는가?
- Spring Boot는 Spring 설정을 어떻게 단순화하는가?
