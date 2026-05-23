# SQLite

## 목차
- SQLite의 특징
- 파일 기반 데이터베이스
- table과 index
- transaction
- 사용 시나리오

## 기초 개념
SQLite는 서버 프로세스 없이 하나의 파일에 데이터를 저장하는 관계형 데이터베이스다. 앱 내장형 저장소, 로컬 개발, 테스트에 자주 쓰인다.

```text
application -> SQLite library -> database file
```

## 간단한 예시
```sql
CREATE TABLE notes (
  id INTEGER PRIMARY KEY,
  title TEXT NOT NULL
);
```

## 반드시 알아야 할 질문
- SQLite는 PostgreSQL 같은 서버형 DB와 무엇이 다른가?
- 파일 기반 DB의 장단점은 무엇인가?
- transaction은 SQLite에서도 왜 중요한가?
- 동시 쓰기에는 어떤 제약이 있는가?
