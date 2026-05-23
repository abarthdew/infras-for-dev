# Spring

## 목차
- [Spring Framework와 Spring Boot](#spring-framework와-spring-boot)
- [IoC와 DI](#ioc와-di)
- [Bean과 ApplicationContext](#bean과-applicationcontext)
- [Web MVC](#web-mvc)
- [Data access](#data-access)
- [Transaction](#transaction)

## Spring Framework와 Spring Boot
Spring Framework는 Java 애플리케이션을 구조화하고 실행하는 프레임워크다. 핵심은 객체 생성, 의존성 연결, 웹 요청 처리, 트랜잭션 같은 반복적인 기반 작업을 프레임워크가 도와준다는 점이다.

Spring Boot는 Spring 애플리케이션을 더 쉽게 시작하고 배포할 수 있게 만든 도구다. 자동 설정, 내장 웹 서버, starter dependency를 제공한다.

```text
Spring Framework  핵심 프레임워크
Spring Boot       설정과 실행을 단순화하는 도구
```

예를 들어 Spring Boot를 쓰면 Tomcat을 따로 설치하지 않아도 내장 서버로 웹 애플리케이션을 실행할 수 있다.

```text
main()
-> SpringApplication.run()
-> embedded Tomcat
-> HTTP request 처리
```

## IoC와 DI
IoC(Inversion of Control)는 객체 생성과 흐름 제어의 주도권이 개발자 코드에서 프레임워크로 넘어가는 구조다. DI(Dependency Injection)는 필요한 의존 객체를 외부에서 주입받는 방식이다.

```java
class OrderService {
  private final OrderRepository repository;

  OrderService(OrderRepository repository) {
    this.repository = repository;
  }
}
```

DI가 테스트와 구조화에 유리한 이유는 구현을 쉽게 바꿀 수 있기 때문이다.

```text
OrderService
-> OrderRepository interface
-> JpaOrderRepository 또는 FakeOrderRepository
```

테스트에서는 실제 DB 대신 fake repository를 넣을 수 있다.

```java
OrderService service = new OrderService(new FakeOrderRepository());
```

DI는 객체가 자기 의존성을 직접 만들지 않게 해서 결합도를 낮춘다.

## Bean과 ApplicationContext
Bean은 Spring이 생성하고 관리하는 객체다. ApplicationContext는 Bean들을 담고 연결하는 Spring 컨테이너다.

```text
ApplicationContext
├── UserController bean
├── UserService bean
└── UserRepository bean
```

Spring은 annotation이나 설정을 보고 Bean을 등록한다.

```java
@Service
class UserService {
}
```

Bean lifecycle은 객체가 생성되고, 의존성이 주입되고, 초기화되고, 종료되는 흐름이다. lifecycle이 중요한 이유는 DB connection, thread pool, file handle 같은 자원을 적절히 열고 닫아야 하기 때문이다.

```text
생성
-> 의존성 주입
-> 초기화
-> 사용
-> 종료
```

## Web MVC
Spring Web MVC는 HTTP 요청을 controller method로 연결하고, 응답을 만들어 반환하는 웹 프레임워크다.

```text
HTTP request
-> DispatcherServlet
-> Controller
-> Service
-> Repository
-> HTTP response
```

예시는 다음과 같다.

```java
@RestController
class UserController {
  @GetMapping("/users/{id}")
  UserResponse getUser(@PathVariable Long id) {
    return new UserResponse(id, "shin");
  }
}
```

Controller는 HTTP 계층을 담당하고, Service는 비즈니스 로직을 담당하며, Repository는 데이터 접근을 담당하는 식으로 역할을 나누는 것이 일반적이다.

```text
Controller  요청/응답 변환
Service     비즈니스 규칙
Repository  DB 접근
```

## Data access
Spring에서 data access는 DB나 외부 저장소에 접근하는 계층을 말한다. JDBC, JPA, MyBatis, Spring Data 같은 도구를 사용할 수 있다.

```text
Service
-> Repository
-> Data access technology
-> Database
```

Spring Data JPA를 사용하면 repository interface만으로 기본 CRUD를 만들 수 있다.

```java
interface UserRepository extends JpaRepository<User, Long> {
  Optional<User> findByEmail(String email);
}
```

data access 계층은 비즈니스 로직과 저장 기술을 분리하는 데 중요하다. Service가 SQL 세부사항을 직접 너무 많이 알면 테스트와 변경이 어려워진다.

## Transaction
Transaction은 여러 DB 작업을 하나의 논리적 단위로 묶는다. Spring에서는 보통 service method에 `@Transactional`을 붙인다.

```java
@Transactional
public void createOrder(CreateOrderCommand command) {
  Order order = orderRepository.save(command.toOrder());
  paymentRepository.save(command.toPayment(order));
}
```

`@Transactional`은 controller보다 service에 붙이는 경우가 많다. 트랜잭션 경계는 HTTP 요청 자체가 아니라 비즈니스 작업 단위에 맞추는 것이 자연스럽기 때문이다.

```text
Controller  요청을 받음
Service     하나의 비즈니스 작업 수행
Repository  DB 변경
```

주의할 점은 Spring의 transaction이 proxy 기반으로 동작한다는 것이다. 같은 객체 내부에서 자기 자신의 transactional method를 직접 호출하면 의도대로 적용되지 않을 수 있다.

```text
transaction boundary = 외부에서 proxy를 통해 호출되는 public method 기준으로 이해
```
