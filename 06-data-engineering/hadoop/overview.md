# Hadoop

## 목차
- [Hadoop의 역할](#hadoop의-역할)
- [HDFS](#hdfs)
- [MapReduce](#mapreduce)
- [YARN](#yarn)
- [Data Locality](#data-locality)
- [Hadoop vs 현대 기술](#hadoop-vs-현대-기술)

---

## Hadoop의 역할

**Hadoop은 수백 테라바이트의 대규모 데이터를 여러 서버에 분산 저장하고 처리하기 위한 오픈소스 생태계**다. Google의 GFS와 MapReduce 논문에서 영감을 받아 설계되었다.

```text
대용량 데이터
    ↓
HDFS (분산 저장)
    ├─ 데이터 1 (서버 A)
    ├─ 데이터 2 (서버 B)
    └─ 데이터 3 (서버 C)
    ↓
MapReduce (분산 처리)
    ├─ Map tasks (병렬)
    ├─ Shuffle (정렬, 그룹화)
    └─ Reduce tasks (집계)
```

---

## HDFS

### "HDFS는 왜 파일을 block으로 나누는가?"

**HDFS는 파일을 block(기본 128MB 또는 256MB)으로 나눠서 여러 서버에 분산 저장함으로써 병렬 처리와 확장성을 확보**한다.**

```text
문제 (전체 저장):
1TB 파일을 1개 서버에만 저장
- 그 서버에서만 처리 가능 (병목)
- 확장 불가능

해결 (block으로 분산):
1TB 파일 → 4개 block (각 256MB)
Block 1 → Server A
Block 2 → Server B
Block 3 → Server C
Block 4 → Server A (복제)

장점:
✓ 병렬 처리 (4개 서버에서 동시 처리)
✓ 복제 (장애 대응)
✓ 스케일 아웃 (서버 추가)
```

### HDFS 아키텍처

```text
NameNode (마스터)
├─ 파일 메타데이터 관리
├─ 파일 위치 정보
└─ 블록 복제 관리

DataNode (워커)
├─ 실제 데이터 저장
├─ 블록 보고
└─ 하트비트 전송
```

### Block 복제

```text
Replication Factor: 3 (기본)
각 block을 3개 복제

Block 1:
├─ Server A (원본)
├─ Server B (복제 1)
└─ Server C (복제 2)

Server A 장애 → 여전히 2개 복사본 있음
```

---

## MapReduce

### "MapReduce는 어떤 처리 모델인가?"

**MapReduce는 대규모 데이터를 병렬로 처리하는 프로그래밍 모델로, Map과 Reduce 두 단계로 구성**된다.

```text
Map (분산):
- 각 block마다 병렬로 map task 실행
- 입력 데이터를 key-value 쌍으로 변환

Example (단어 수 세기):
Input: ["hello world", "hello hadoop"]
Map output: [("hello", 1), ("world", 1), ("hello", 1), ("hadoop", 1)]

Shuffle (정렬, 그룹화):
같은 key끼리 묶음
("hello", [1, 1])
("world", [1])
("hadoop", [1])

Reduce (집계):
각 key별로 reduce task 실행
("hello", 2)
("world", 1)
("hadoop", 1)
```

### Map-Reduce 코드

```python
# Hadoop Streaming (Python)

# mapper.py
import sys

for line in sys.stdin:
    words = line.strip().split()
    for word in words:
        print(f"{word}\t1")

# reducer.py
import sys

current_word = None
count = 0

for line in sys.stdin:
    word, cnt = line.strip().split("\t")
    
    if word == current_word:
        count += int(cnt)
    else:
        if current_word:
            print(f"{current_word}\t{count}")
        current_word = word
        count = int(cnt)

if current_word:
    print(f"{current_word}\t{count}")
```

---

## YARN

### YARN (Yet Another Resource Negotiator)

**YARN은 Hadoop 클러스터의 자원(CPU, 메모리)을 관리하고 여러 애플리케이션을 스케줄하는 리소스 매니저**다.

```text
YARN 이전 (고정 역할):
- TaskTracker: map/reduce만 실행

YARN 이후 (유연한 역할):
- NodeManager: 모든 종류의 task 실행
- MapReduce
- Spark
- Hive
- Presto
```

---

## Data Locality

### "data locality는 왜 중요했는가?"

**Data Locality는 "데이터가 있는 곳에서 처리"하는 원칙으로, 네트워크 비용을 절감**한다.**

```text
Data Locality 없이 (비효율):
Server A의 data
    ↓ (네트워크 전송)
Server B에서 처리
    ↓ (결과 전송)
→ 네트워크 대역폭 낭비

Data Locality (효율):
Server A의 data
    ↓ (같은 서버)
Server A에서 처리
    ↓ (로컬 저장)
→ 네트워크 불필요

Hadoop의 전략:
1. Rack locality: 같은 랙의 서버 선택
2. Node locality: 같은 노드 선택 (최고 우선)
3. Off-rack: 다른 랙 (마지막 수단)
```

---

## Hadoop vs 현대 기술

### "Hadoop은 현대 데이터 플랫폼에서 어떤 위치인가?"

**Hadoop은 여전히 유효하지만, Spark, Cloud Storage와 함께 평가해야 한다.**

```text
Hadoop (2006-2016):
✓ 장점: 오픈소스, 강력한 분산 처리
✗ 단점: 느림 (Map → Shuffle → Reduce 3단계), 운영 복잡

Spark (2010-):
✓ 장점: 빠름 (메모리 기반), SQL 지원, 다양한 API
✓ 같은 YARN 위에서 실행 가능
✗ 단점: 메모리 사용 많음

Cloud Storage + Serverless (2015-):
✓ 장점: 관리 부담 없음, 저렴, 스케일 자유로움
✓ 예: S3 + Spark + BigQuery
✗ 단점: vendor lock-in

현재 추세:
- Hadoop HDFS: 감소 중 (Cloud Storage로 이전)
- Spark: 계속 성장 (Hadoop과 독립적으로)
- YARN: 레거시 (Kubernetes로 교체)
```

### 선택 기준

```text
Hadoop MapReduce:
- 매우 큰 배치 (100TB+)
- 온프레미스
- 기존 Hadoop 투자

Spark:
- 대부분의 빅데이터 처리
- 빠른 개발 필요
- 반복 처리

Cloud 서비스:
- 관리 부담 최소화
- 비용 최적화
- 스케일링 자유도
```

---

## 정리

| 개념 | 설명 |
|------|------|
| HDFS | 분산 파일 시스템 (block 기반) |
| MapReduce | Map + Shuffle + Reduce 처리 모델 |
| YARN | 리소스 관리 및 스케줄러 |
| Data Locality | 데이터 위치에서 처리 |

---

## Related Notes

- [Batch Processing](../batch-processing/overview.md)
- [Data Pipeline](../data-pipeline/overview.md)
