# Hadoop

## 목차
- HDFS
- MapReduce
- YARN
- batch processing
- data locality

## 기초 개념
Hadoop은 큰 데이터를 여러 서버에 분산 저장하고 처리하기 위한 생태계다. 현대에는 Spark, cloud storage와 함께 비교해서 이해하는 것이 좋다.

```text
large data -> HDFS -> distributed processing
```

## 간단한 예시
```text
input files
-> map tasks
-> shuffle
-> reduce tasks
-> output
```

## 반드시 알아야 할 질문
- HDFS는 왜 파일을 block으로 나누는가?
- MapReduce는 어떤 처리 모델인가?
- data locality는 왜 중요했는가?
- Hadoop은 현대 데이터 플랫폼에서 어떤 위치인가?
