# Elasticsearch

## 목차
- [검색 엔진의 역할](#검색-엔진의-역할)
- [Index와 Document](#index와-document)
- [Inverted Index](#inverted-index)
- [Analyzer](#analyzer)
- [Full-text Search](#full-text-search)
- [Aggregation](#aggregation)

---

## 검색 엔진의 역할

Elasticsearch는 **검색과 분석에 특화된 분산 검색 엔진**이다. 텍스트를 분석해 인덱스를 만들고, 빠른 전문 검색과 실시간 집계를 제공한다.

### 관계형 DB 검색과의 차이

**관계형 DB 검색:**

```sql
-- LIKE로 전문 검색하면 느림
SELECT * FROM articles 
WHERE content LIKE '%linux network%';
-- 전체 행을 스캔해야 함 (풀 스캔)
```

**Elasticsearch 검색:**

```json
// 인덱싱된 데이터에서 빠르게 검색
GET /articles/_search
{
  "query": {
    "match": {
      "content": "linux network"
    }
  }
}
```

Elasticsearch는 "linux", "network" 같은 단어가 어느 문서에 있는지 미리 인덱싱해두므로, LIKE 검색보다 훨씬 빠르다.

### 사용 시나리오

```text
로그 분석          수백만 개의 로그 라인에서 특정 에러 검색
검색 엔진          제품명, 설명으로 상품 검색
시계열 데이터      메트릭, 추적 데이터 수집 및 분석
원본 DB 아님       읽기 전용 인덱스 (원본은 관계형 DB)
```

Elasticsearch는 원본 데이터베이스를 **완전히 대체하지 않는다**. 대신 빠른 검색과 분석을 위해 관계형 DB 위에 구축된다.

```text
관계형 DB (원본)
    ↓ (데이터 동기화)
Elasticsearch (인덱스)
    ↓
빠른 검색 제공
```

---

## Index와 Document

Elasticsearch의 기본 구조는 **Index(인덱스)**에 **Document(문서)**를 저장하는 것이다.

### Index

Index는 관계형 DB의 **테이블**에 해당한다. 문서들의 집합이다.

```
Elasticsearch          관계형 DB
-----------------      -----------
Index                  Table
Document               Row
Field                  Column
```

### Document

각 Document는 JSON 형식의 데이터다.

```json
{
  "_id": "1",
  "_index": "articles",
  "_type": "_doc",
  "title": "Linux Networking Basics",
  "content": "Learn about network packets, IP, TCP...",
  "author": "john",
  "tags": ["linux", "network", "beginner"],
  "published_date": "2026-05-23",
  "view_count": 1234
}
```

### Index 생성 및 Document 저장

```bash
# Index 생성
PUT /articles
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}

# Document 저장 (자동 ID 생성)
POST /articles/_doc
{
  "title": "Linux Networking Basics",
  "content": "Learn about network packets...",
  "author": "john",
  "tags": ["linux", "network"]
}

# Document 저장 (ID 지정)
PUT /articles/_doc/1
{
  "title": "Elasticsearch Basics",
  "content": "How to use Elasticsearch for searching...",
  "author": "jane"
}

# Document 조회
GET /articles/_doc/1

# Document 수정
POST /articles/_doc/1/_update
{
  "doc": {
    "view_count": 5000
  }
}

# Document 삭제
DELETE /articles/_doc/1
```

### Mapping (스키마)

Mapping은 각 필드의 데이터 타입을 정의한다 (SQL의 CREATE TABLE과 유사).

```json
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "content": {
        "type": "text"
      },
      "author": {
        "type": "keyword"  // 분석하지 않음 (정확히 일치)
      },
      "tags": {
        "type": "keyword"
      },
      "published_date": {
        "type": "date"
      },
      "view_count": {
        "type": "integer"
      }
    }
  }
}
```

**필드 타입:**
- `text`: 전문 검색 (분석됨, 토큰화)
- `keyword`: 정확한 매칭 (분석 안 됨)
- `integer`, `long`: 숫자
- `date`: 날짜
- `object`: 중첩된 객체
- `array`: 배열

---

## Inverted Index

**Inverted Index(역색인)**는 Elasticsearch가 빠른 검색을 제공하는 핵심 이유다.

### 일반 인덱스 vs 역색인

**일반 인덱스 (Document → Term):**

```
Document 1: "linux network basics"
Document 2: "network security guide"
Document 3: "linux kernel design"
```

"network"를 찾으려면 모든 문서를 스캔해야 한다.

**역색인 (Term → Document):**

```
linux    → Document 1, Document 3
network  → Document 1, Document 2
basics   → Document 1
security → Document 2
guide    → Document 2
kernel   → Document 3
design   → Document 3
```

"network"를 검색하면 역색인에서 바로 [Document 1, Document 2]를 찾을 수 있다. (매우 빠름)

### 역색인 구성 과정

```text
Document 입력
    ↓
Analyzer가 토큰화 및 정규화
    ↓ (예: "Linux NETWORK Basics" → ["linux", "network", "basics"])
역색인 생성
    ↓
term: "linux"     → [doc_id: 1, doc_id: 3, ...]
term: "network"   → [doc_id: 1, doc_id: 2, ...]
term: "basics"    → [doc_id: 1, ...]
```

역색인이 있으면 검색이 O(1) ~ O(log n) 시간에 가능하다. (DB LIKE는 O(n))

---

## Analyzer

**Analyzer(분석기)**는 텍스트를 토큰(단어)으로 나누고 정규화하는 도구다.

### Analyzer 구성

```text
Character Filter (문자 필터)
    ↓ (특수 문자 제거, HTML 태그 제거 등)
Tokenizer (토큰화)
    ↓ (공백, 구두점 기준으로 단어 분리)
Token Filter (토큰 필터)
    ↓ (소문자 변환, 불용어 제거, 형태소 분석 등)
```

### 기본 Analyzer

```json
// Standard Analyzer (기본값)
GET /articles/_analyze
{
  "analyzer": "standard",
  "text": "The quick brown fox jumps!"
}

// 결과
["the", "quick", "brown", "fox", "jumps"]
```

### 언어별 Analyzer

**English Analyzer (영어):**

```json
GET /articles/_analyze
{
  "analyzer": "english",
  "text": "running runs runner"
}

// 결과 (형태소 분석: 어간 추출)
["run", "run", "runner"]
```

**Korean Analyzer (한국어, 플러그인 필요):**

```json
GET /articles/_analyze
{
  "analyzer": "korean",
  "text": "엘라스틱서치는 검색 엔진입니다"
}

// 결과 (형태소 분석)
["엘라스틱", "서치", "검색", "엔진"]
```

### 커스텀 Analyzer

```json
PUT /articles
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": ["lowercase", "stop"]  // 불용어 제거
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

---

## Full-text Search

**Full-text Search(전문 검색)**는 문서 내 단어를 검색하는 기능이다.

### 기본 검색

```json
// match: 텍스트를 분석해서 검색
GET /articles/_search
{
  "query": {
    "match": {
      "title": "linux network"
    }
  }
}

// 결과: "linux"나 "network"를 포함하는 문서
```

### 검색 연산자

```json
// AND (둘 다 포함)
GET /articles/_search
{
  "query": {
    "match": {
      "title": {
        "query": "linux network",
        "operator": "and"  // 기본값은 "or"
      }
    }
  }
}

// Phrase Search (정확한 구문)
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "content": "network packets"  // "network packets" 순서대로
    }
  }
}

// Boolean 쿼리 (복합 조건)
GET /articles/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "linux" } }
      ],
      "must_not": [
        { "match": { "tags": "advanced" } }
      ],
      "filter": [
        { "range": { "published_date": { "gte": "2026-01-01" } } }
      ]
    }
  }
}
```

### 검색 결과

```json
{
  "hits": {
    "total": { "value": 2 },
    "hits": [
      {
        "_id": "1",
        "_score": 1.234,  // 관련도 점수 (높을수록 관련도 높음)
        "_source": {
          "title": "Linux Networking Basics",
          "content": "..."
        }
      },
      {
        "_id": "2",
        "_score": 0.567,
        "_source": { ... }
      }
    ]
  }
}
```

---

## Aggregation

**Aggregation(집계)**는 데이터를 분석하고 통계를 내는 기능이다. SQL의 GROUP BY와 유사하다.

### 기본 집계

**Terms Aggregation (카테고리별 개수):**

```json
GET /articles/_search
{
  "size": 0,  // 문서 반환하지 않음 (집계만)
  "aggs": {
    "tags_count": {
      "terms": {
        "field": "tags",
        "size": 10  // 상위 10개
      }
    }
  }
}

// 결과
{
  "aggregations": {
    "tags_count": {
      "buckets": [
        {
          "key": "linux",
          "doc_count": 45
        },
        {
          "key": "network",
          "doc_count": 38
        },
        ...
      ]
    }
  }
}
```

**Date Histogram (시간대별 분석):**

```json
GET /articles/_search
{
  "aggs": {
    "articles_per_day": {
      "date_histogram": {
        "field": "published_date",
        "calendar_interval": "day"
      }
    }
  }
}

// 결과: 일별 문서 개수
```

**Range Aggregation (범위별 분석):**

```json
GET /articles/_search
{
  "aggs": {
    "views_range": {
      "range": {
        "field": "view_count",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 1000 },
          { "from": 1000 }
        ]
      }
    }
  }
}
```

**Avg/Sum (평균/합계):**

```json
GET /articles/_search
{
  "aggs": {
    "average_views": {
      "avg": {
        "field": "view_count"
      }
    },
    "total_views": {
      "sum": {
        "field": "view_count"
      }
    }
  }
}
```

### 중첩 집계

```json
GET /articles/_search
{
  "aggs": {
    "by_author": {
      "terms": {
        "field": "author"
      },
      "aggs": {
        "average_views": {
          "avg": {
            "field": "view_count"
          }
        }
      }
    }
  }
}

// 결과: 작성자별 평균 조회수
```

---

## 정리

**Elasticsearch의 특징:**

1. **빠른 검색**: 역색인으로 대규모 데이터에서 밀리초 단위 검색
2. **언어 지원**: 언어별 Analyzer로 다양한 텍스트 처리
3. **분석 기능**: 집계를 통해 즉각적인 인사이트 도출
4. **분산 구조**: 대규모 데이터를 여러 노드에 분산 저장

**사용 시 주의:**

- **원본 DB 아님**: 항상 원본 데이터는 관계형 DB에 유지
- **읽기 최적화**: 쓰기는 느릴 수 있음 (인덱싱 비용)
- **메모리 사용**: 인덱싱은 메모리/디스크를 많이 사용
- **일관성**: 검색 결과가 즉시 반영되지 않을 수 있음 (refresh interval)
