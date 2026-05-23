# JavaScript

## 목차
- 값과 타입
- function과 closure
- prototype
- event loop
- Promise와 async/await
- browser와 Node.js 런타임

## 기초 개념
JavaScript는 웹 브라우저와 서버 런타임에서 모두 쓰이는 동적 타입 언어다. 비동기 I/O와 이벤트 기반 프로그래밍을 이해하는 것이 중요하다.

```text
call stack -> event loop -> task queue
```

## 간단한 예시
```javascript
async function main() {
  const res = await fetch("https://example.com");
  console.log(res.status);
}
```

## 반드시 알아야 할 질문
- closure는 왜 중요한가?
- `this`는 언제 달라지는가?
- Promise는 callback 문제를 어떻게 완화하는가?
- 브라우저 JS와 Node.js는 무엇이 다른가?
