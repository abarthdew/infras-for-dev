# Java

## 목차
- [JVM과 bytecode](#jvm과-bytecode)
- [class와 object](#class와-object)
- [interface와 inheritance](#interface와-inheritance)
- [collection](#collection)
- [exception](#exception)
- [concurrency](#concurrency)

## JVM과 bytecode
Java는 JVM(Java Virtual Machine) 위에서 실행되는 언어다. Java 소스 코드는 바로 운영체제가 실행하는 기계어가 아니라 bytecode로 컴파일된다.

```text
Java source
-> javac
-> bytecode(.class)
-> JVM
-> OS와 CPU
```

JVM은 운영체제 위에서 실행되는 런타임이다. 그래서 Java 프로그램은 JVM이 설치된 여러 운영체제에서 같은 bytecode를 실행할 수 있다.

```text
Windows JVM
Linux JVM
macOS JVM
-> 같은 .class 실행 가능
```

JVM은 단순 실행기만이 아니다. 메모리 관리, garbage collection, class loading, JIT compilation 같은 기능을 제공한다.

```text
class loader  class 파일 로딩
JIT compiler  자주 실행되는 bytecode를 native code로 최적화
GC            더 이상 참조되지 않는 객체 메모리 회수
```

## class와 object
class는 객체를 만들기 위한 설계도이고, object는 그 설계도로 만들어진 실제 값이다.

```java
class User {
  String name;

  User(String name) {
    this.name = name;
  }
}
```

```java
User user = new User("shin");
```

위 코드에서 `User`는 class이고, `new User("shin")`으로 만들어진 값은 object다.

Java의 타입은 크게 primitive type과 reference type으로 나눌 수 있다.

```text
primitive type  int, long, boolean 같은 값 자체
reference type  객체를 가리키는 참조
```

예를 들어 `int x = 10`은 값 자체를 다루고, `User user`는 객체를 직접 담는 것이 아니라 객체를 가리키는 참조를 담는다.

```text
user variable -> User object
```

이 차이는 `null`, 메모리, equality를 이해할 때 중요하다.

## interface와 inheritance
inheritance는 부모 class의 속성과 동작을 자식 class가 물려받는 것이다. interface는 어떤 기능을 제공해야 하는지 약속하는 타입이다.

```java
interface Repository {
  void save(String value);
}
```

```java
class MemoryRepository implements Repository {
  public void save(String value) {
    System.out.println(value);
  }
}
```

interface가 필요한 이유는 구현을 바꿔도 사용하는 쪽 코드를 안정적으로 유지하기 위해서다.

```text
Service -> Repository interface -> MemoryRepository
                              -> DatabaseRepository
```

상속은 코드 재사용에 유용하지만 부모-자식 결합을 강하게 만든다. Java에서는 구현 상속보다 interface를 통한 역할 분리를 더 자주 권장한다.

```text
inheritance  is-a 관계와 코드 재사용
interface    역할과 계약 정의
```

## collection
collection은 여러 값을 담는 자료구조다. Java에서 자주 쓰는 collection은 `List`, `Set`, `Map`이다.

```text
List  순서가 있고 중복 허용
Set   중복을 허용하지 않음
Map   key-value 저장
```

예시는 다음과 같다.

```java
List<String> names = List.of("kim", "lee", "kim");
Set<String> uniqueNames = Set.of("kim", "lee");
Map<Long, String> users = Map.of(1L, "shin");
```

collection을 고를 때는 조회 방식과 중복 허용 여부를 생각해야 한다.

```text
순서가 필요한가?
중복을 허용하는가?
key로 빠르게 찾을 것인가?
```

예를 들어 ID로 사용자를 찾는 구조라면 `List<User>`보다 `Map<Long, User>`가 더 적절할 수 있다.

## exception
exception은 프로그램 실행 중 발생한 예외 상황을 표현하는 객체다.

```java
try {
  int result = 10 / 0;
} catch (ArithmeticException e) {
  System.out.println("cannot divide by zero");
}
```

Java exception은 checked exception과 unchecked exception으로 나눌 수 있다.

```text
checked exception    컴파일러가 처리 여부를 요구
unchecked exception  RuntimeException 계열, 컴파일러가 강제하지 않음
```

checked exception의 장점은 호출자가 실패 가능성을 명시적으로 다루게 한다는 점이다. 단점은 코드가 장황해지고, 의미 없이 `throws`가 전파될 수 있다는 점이다.

예외는 정상적인 조건 분기용으로 남용하면 안 된다. 예외는 보통 파일 없음, 네트워크 실패, 잘못된 상태처럼 예외적인 흐름을 표현하는 데 적합하다.

## concurrency
concurrency는 여러 작업을 같은 시간대에 진행하는 것처럼 다루는 개념이다. Java에서는 `Thread`, `ExecutorService`, `CompletableFuture`, synchronized, lock, atomic class 등을 사용한다.

```java
Thread thread = new Thread(() -> {
  System.out.println("worker");
});
thread.start();
```

동시성에서 가장 중요한 문제는 공유 상태다. 여러 스레드가 같은 값을 동시에 수정하면 race condition이 생길 수 있다.

```text
thread A -> count 읽기
thread B -> count 읽기
thread A -> count + 1 저장
thread B -> count + 1 저장
```

이 경우 두 번 증가해야 하는 값이 한 번만 증가할 수 있다. 그래서 synchronized나 atomic type을 사용한다.

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();
```

Java 동시성은 "스레드를 많이 만들면 빠르다"가 아니라, 작업 특성과 공유 상태를 이해하고 안전하게 조정하는 문제다.
