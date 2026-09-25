# 예제로 보는 마이크로서비스 배포 설계

간단한 투표 애플리케이션(voting-app)을 예시로, 여러 컴포넌트로 이루어진 앱을 Kubernetes에 배포할 때 어떤 점을 먼저 고민해야 하는지 정리한다.

## 목차
- [예제 애플리케이션 구조](#예제-애플리케이션-구조)
- [배포 전 설계 질문](#배포-전-설계-질문)
- [연결 요구사항 정리](#연결-요구사항-정리)
- [서비스 이름을 신중하게 정해야 하는 이유](#서비스-이름을-신중하게-정해야-하는-이유)
- [정리](#정리)

---

## 예제 애플리케이션 구조

- **voting-app** (Python): 사용자가 두 선택지 중 하나에 투표하는 웹 인터페이스. 포트 80 리스닝.
- **redis**: 투표 결과를 저장하는 인메모리 DB. 포트 6379.
- **worker** (.NET): Redis의 새 투표를 읽어 PostgreSQL에 반영. 리스닝 포트 없음 — 아무도 worker에 직접 접근하지 않음.
- **postgresql (db)**: 카테고리별 투표 수를 저장하는 영구 DB. 포트 5432.
- **result-app** (Node.js): PostgreSQL에서 집계 결과를 읽어 사용자에게 보여주는 웹 인터페이스. 포트 80 리스닝.

## 배포 전 설계 질문

Kubernetes에 배포하기 전에 먼저 답해야 하는 것들:
1. 이 컴포넌트들을 무엇으로 배포할 것인가 (Pod 단독? ReplicaSet? Deployment?)
2. 컴포넌트 간 연결성을 어떻게 확보할 것인가 — 어떤 컴포넌트가 어떤 서비스에 접근해야 하는지 명확히 정리되어야 함.
3. 외부 사용자에게 노출해야 하는 컴포넌트는 무엇인가.

컨테이너는 Kubernetes 클러스터에 직접 배포할 수 없으므로, 가장 작은 배포 단위인 **Pod**로 우선 감싸야 한다(단순화를 위해 이 예제는 Pod 단독으로 배포하지만, 실무에서는 Deployment로 감싸는 것이 일반적이다).

## 연결 요구사항 정리

| 컴포넌트 | 접근 대상 | 서비스 타입 |
|---|---|---|
| voting-app | redis (저장) | ClusterIP |
| worker | redis (읽기), postgresql (쓰기) | ClusterIP |
| result-app | postgresql (읽기) | ClusterIP |
| voting-app, result-app | 외부 사용자 | NodePort |
| worker | (없음 — 아무도 worker에 접근하지 않음) | 서비스 불필요 |

- Pod IP는 재시작 시 바뀔 수 있으므로 IP로 직접 연결하면 안 되고, 반드시 **Service**를 매개로 연결한다.
- 총 5개 파드에 4개 서비스(내부용 redis/db + 외부용 voting-app/result-app)가 필요하며, worker는 어떤 서비스도 필요하지 않다 — 다른 컴포넌트나 외부 사용자가 worker에 직접 접근하는 경로가 없기 때문이다.

## 서비스 이름을 신중하게 정해야 하는 이유

- voting-app과 worker의 소스 코드는 Redis DB 호스트명을 `redis`로 하드코딩하고 있다. 따라서 Redis Pod를 노출하는 Service의 이름도 반드시 **`redis`**여야 한다.
- 마찬가지로 worker와 result-app은 DB 호스트명을 `db`로 찾도록 되어 있으므로, PostgreSQL Service의 이름은 **`db`**여야 한다.
- (참고: 소스 코드에 호스트명을 하드코딩하는 것은 권장되는 방식이 아니며 환경 변수 등을 쓰는 것이 바람직하지만, 예제 단순화를 위해 그대로 따른다.)
- DB 접속용 사용자명/비밀번호도 애플리케이션 코드에 `postgres`/`postgres`로 고정되어 있으므로, DB Pod를 생성할 때 해당 초기 자격 증명으로 맞춰줘야 한다 — 실습/예제 목적의 기본값이며 운영 환경에는 그대로 쓰면 안 된다.

## 정리

- 총 5개 Pod, 4개 Service(2개는 내부용 ClusterIP, 2개는 외부용 NodePort)로 요약된다.
- Service가 필요 없는 컴포넌트(worker)를 구분하는 기준은 단순하다 — **다른 컴포넌트나 외부 사용자가 그 컴포넌트에 접근해야 하는가?** 아니라면 Service는 불필요하다.
- 이 설계 순서(컴포넌트 나열 → 연결 요구사항 정리 → 외부 노출 대상 정리 → Pod/Service 개수 산정)는 실제 마이크로서비스 앱을 Kubernetes로 옮길 때 그대로 적용할 수 있는 체크리스트다.

# 색인과 출처
- Kubernetes for the Absolute Beginners (KodeKloud/Udemy) — MicroServices Architecture 섹션
