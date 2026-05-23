# JavaScript

## 목차
- [값과 타입](#값과-타입)
- [function과 closure](#function과-closure)
- [prototype](#prototype)
- [event loop](#event-loop)
- [Promise와 async/await](#promise와-asyncawait)
- [browser와 Node.js 런타임](#browser와-nodejs-런타임)

## 값과 타입
JavaScript는 동적 타입 언어다. 변수에 타입이 고정되는 것이 아니라 값이 타입을 가진다.

```javascript
let value = 1;
value = "hello";
```

기본 타입은 다음처럼 볼 수 있다.

```text
number
string
boolean
undefined
null
symbol
bigint
object
```

`null`과 `undefined`는 구분해야 한다.

```text
undefined  값이 아직 할당되지 않음
null       의도적으로 비어 있음을 표현
```

객체, 배열, 함수는 reference로 다뤄진다.

```javascript
const a = { name: "shin" };
const b = a;
b.name = "kim";
console.log(a.name); // kim
```

이 예시는 `a`와 `b`가 같은 객체를 가리킨다는 점을 보여준다.

## function과 closure
JavaScript에서 function은 값처럼 변수에 담고, 인자로 넘기고, 반환할 수 있다.

```javascript
function add(a, b) {
  return a + b;
}
```

closure는 함수가 자신이 만들어진 lexical scope의 변수를 기억하는 성질이다.

```javascript
function makeCounter() {
  let count = 0;

  return function increment() {
    count += 1;
    return count;
  };
}

const counter = makeCounter();
counter(); // 1
counter(); // 2
```

`makeCounter` 실행이 끝났는데도 `increment` 함수는 `count`를 기억한다. 이것이 closure다.

closure가 중요한 이유는 상태를 은닉하거나, callback이 만들어진 시점의 context를 유지할 수 있기 때문이다.

## prototype
JavaScript 객체는 prototype을 통해 다른 객체의 속성과 메서드를 찾아갈 수 있다. class 문법도 내부적으로는 prototype 기반 모델 위에 있다.

```javascript
const user = {
  name: "shin",
};

console.log(user.toString);
```

`user` 객체에 `toString`이 직접 없어도 prototype chain을 따라가며 찾는다.

```text
user
-> Object.prototype
-> null
```

class 문법은 더 익숙한 형태로 객체 생성과 메서드 정의를 제공한다.

```javascript
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `hello ${this.name}`;
  }
}
```

prototype을 이해하면 method 공유, inheritance, `this` 동작을 더 잘 이해할 수 있다.

## event loop
JavaScript는 보통 하나의 call stack에서 코드를 실행한다. 하지만 브라우저와 Node.js는 비동기 작업을 런타임에 맡기고, 완료된 작업의 callback을 queue에 넣어 event loop가 처리하게 한다.

```text
call stack
runtime APIs
task queue / microtask queue
event loop
```

예를 들어 `setTimeout`은 즉시 실행되는 것이 아니라, 지정 시간이 지난 뒤 callback이 queue에 들어간다.

```javascript
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

출력은 보통 다음과 같다.

```text
A
C
B
```

event loop는 비동기 I/O와 UI 반응성을 이해하는 핵심이다. 긴 CPU 작업이 call stack을 오래 점유하면 비동기 코드가 있어도 화면이나 이벤트 처리가 막힐 수 있다.

## Promise와 async/await
Promise는 비동기 작업의 성공 또는 실패 결과를 표현하는 객체다. callback 중첩을 줄이고 비동기 흐름을 값처럼 다룰 수 있게 한다.

```javascript
fetch("https://example.com")
  .then((res) => res.text())
  .then((body) => console.log(body))
  .catch((err) => console.error(err));
```

`async/await`는 Promise를 더 동기 코드처럼 읽히게 만드는 문법이다.

```javascript
async function main() {
  try {
    const res = await fetch("https://example.com");
    const body = await res.text();
    console.log(body);
  } catch (err) {
    console.error(err);
  }
}
```

Promise가 callback 문제를 완전히 없애는 것은 아니지만, 에러 처리와 순차/병렬 비동기 조합을 더 명확하게 만든다.

```javascript
const [user, orders] = await Promise.all([
  fetchUser(),
  fetchOrders(),
]);
```

## browser와 Node.js 런타임
JavaScript 언어 자체와 런타임은 구분해야 한다. 브라우저와 Node.js는 같은 JavaScript 언어를 실행하지만 제공하는 API가 다르다.

```text
JavaScript language  문법과 기본 객체
Browser runtime      DOM, fetch, localStorage, window
Node.js runtime      fs, path, process, server API
```

브라우저 JavaScript는 화면과 사용자 이벤트를 다룬다.

```javascript
document.querySelector("button").addEventListener("click", () => {
  console.log("clicked");
});
```

Node.js는 서버와 파일 시스템 작업에 자주 쓰인다.

```javascript
import { readFile } from "node:fs/promises";

const text = await readFile("README.md", "utf8");
```

브라우저와 Node.js의 차이를 모르면 `document is not defined`나 `fs is not available` 같은 오류를 만나기 쉽다.
