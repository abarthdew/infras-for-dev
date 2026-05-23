# Java

## 목차
- JVM과 bytecode
- class와 object
- interface와 inheritance
- collection
- exception
- concurrency

## 기초 개념
Java는 JVM 위에서 실행되는 정적 타입 객체지향 언어다. 소스 코드는 bytecode로 컴파일되고 JVM이 이를 실행한다.

```text
Java source -> bytecode -> JVM -> OS
```

## 간단한 예시
```java
public class Main {
  public static void main(String[] args) {
    System.out.println("hello");
  }
}
```

## 반드시 알아야 할 질문
- JVM은 운영체제와 어떤 관계인가?
- primitive type과 reference type은 무엇이 다른가?
- interface는 왜 필요한가?
- checked exception은 어떤 장단점이 있는가?
