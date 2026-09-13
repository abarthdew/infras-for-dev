# Python

## 목차
- [값과 타입](#값과-타입)
- [function과 module](#function과-module)
- [list, dict, tuple](#list-dict-tuple)
- [exception](#exception)
- [virtual environment](#virtual-environment)
- [iterator와 generator](#iterator와-generator)

## 값과 타입
Python은 동적 타입 언어다. 변수에 타입을 선언하지 않아도 실행 시점의 값이 타입을 가진다.

```python
value = 1
value = "hello"
```

동적 타입의 장점은 코드 작성이 빠르고 유연하다는 점이다. 단점은 타입 관련 오류가 실행 전까지 드러나지 않을 수 있다는 점이다.

```text
장점  간결함, 빠른 실험, 유연한 데이터 처리
단점  런타임 오류 가능성, 큰 코드베이스에서 추론 어려움
```

Python의 주요 기본 타입은 다음과 같다.

```text
int
float
str
bool
list
dict
tuple
set
None
```

최근 Python에서는 type hint를 사용해 가독성과 도구 지원을 높인다.

```python
def add(a: int, b: int) -> int:
    return a + b
```

type hint는 기본적으로 런타임 타입 검사를 강제하지 않는다. 사람이 읽고, IDE와 type checker가 분석하는 힌트에 가깝다.

## function과 module
function은 재사용 가능한 코드 블록이다.

```python
def greet(name: str) -> str:
    return f"hello {name}"
```

Python 함수는 값처럼 다룰 수 있다. 변수에 담거나 다른 함수에 전달할 수 있다.

```python
def apply(func, value):
    return func(value)

apply(str.upper, "hello")
```

module은 Python 파일 하나를 import 가능한 단위로 보는 개념이다.

```text
math_utils.py
-> import math_utils
```

패키지는 여러 module을 디렉토리로 묶은 구조다.

```text
my_package/
├── __init__.py
├── user.py
└── order.py
```

import는 단순 복사 붙여넣기가 아니라, Python이 module을 찾아 실행하고 module object를 현재 코드에서 참조하게 만드는 과정이다.

## list, dict, tuple
`list`는 순서가 있는 가변 sequence다.

```python
names = ["kim", "lee", "park"]
names.append("shin")
```

`dict`는 key-value를 저장하는 mapping이다.

```python
user = {
    "id": 1,
    "name": "shin",
}
```

`tuple`은 순서가 있는 불변 sequence다.

```python
point = (10, 20)
```

선택 기준은 다음과 같다.

```text
list   순서 있는 여러 값, 변경 가능
dict   key로 값을 빠르게 찾음
tuple  변경하지 않을 묶음
```

Python 자료구조는 매우 강력하지만, mutable object를 공유할 때는 조심해야 한다.

```python
a = []
b = a
b.append(1)
print(a)  # [1]
```

## exception
exception은 실행 중 발생한 예외 상황을 표현한다. 파일이 없거나, 네트워크가 실패하거나, 잘못된 값이 들어오는 경우에 사용한다.

```python
try:
    with open("missing.txt", "r", encoding="utf-8") as f:
        text = f.read()
except FileNotFoundError:
    text = ""
```

예외를 무조건 잡아서 숨기면 안 된다. 오류를 숨기면 문제 원인을 찾기 어려워진다.

```python
try:
    risky_work()
except Exception:
    pass
```

위 패턴은 특별한 이유가 없다면 피하는 것이 좋다. 어떤 오류를 처리할지 구체적으로 잡고, 필요한 경우 로그를 남겨야 한다.

```text
예외 처리의 목적
-> 복구 가능한 오류 처리
-> 사용자에게 적절한 메시지 제공
-> 시스템 상태 정리
```

## virtual environment
virtual environment는 프로젝트별 Python 패키지 설치 공간을 분리하는 기능이다.

```bash
python -m venv .venv
```

프로젝트마다 필요한 패키지 버전이 다를 수 있다.

```text
project A -> Django 4
project B -> Django 5
```

전역 Python 환경에 모두 설치하면 버전 충돌이 생길 수 있다. virtual environment는 이런 충돌을 줄인다.

```bash
.venv\Scripts\activate
pip install requests
```

의존성은 보통 파일로 기록한다.

```bash
pip freeze > requirements.txt
```

정리하면 virtual environment는 Python 실행 자체를 바꾸는 것이 아니라, 해당 프로젝트에서 사용할 interpreter와 package 경로를 분리하는 장치다.

## iterator와 generator
iterator는 값을 하나씩 꺼낼 수 있는 객체다.

```python
numbers = iter([1, 2, 3])
print(next(numbers))
print(next(numbers))
```

generator는 iterator를 쉽게 만들 수 있는 함수다. `yield`를 사용한다.

```python
def count_up_to(n):
    current = 1
    while current <= n:
        yield current
        current += 1
```

generator는 모든 값을 한 번에 list로 만들지 않고 필요할 때 하나씩 생성한다.

```python
for number in count_up_to(3):
    print(number)
```

일반 list와 generator의 차이는 메모리와 계산 시점이다.

```text
list       모든 값을 메모리에 보관
generator  필요할 때 하나씩 생성
```

큰 파일을 한 줄씩 읽거나, 무한 sequence를 표현하거나, pipeline 처리할 때 generator가 유용하다.
