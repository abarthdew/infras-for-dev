# Kubernetes 아키텍처 상세

## 목차
- [컨테이너와 컨테이너 오케스트레이션](#컨테이너와-컨테이너-오케스트레이션)
- [Master와 Worker 컴포넌트 개요](#master와-worker-컴포넌트-개요)
- [ETCD](#etcd)
- [kube-apiserver](#kube-apiserver)
- [kube-controller-manager](#kube-controller-manager)
- [kube-scheduler](#kube-scheduler)
- [kubelet](#kubelet)
- [kube-proxy](#kube-proxy)
- [kubectl](#kubectl)
- [Namespace](#namespace)
- [Imperative vs Declarative](#imperative-vs-declarative)

---

## 컨테이너와 컨테이너 오케스트레이션

서로 다른 애플리케이션을 개발할 때 OS·라이브러리·의존성 호환성 문제가 발생하기 쉽다. Docker는 각 요소를 컨테이너로 분리해 호환성 문제 없이 간단한 명령으로 구동할 수 있게 한다.

- **컨테이너**: 완전히 고립된 환경. 가상 머신처럼 자체 프로세스·네트워크 인터페이스가 있지만, 모든 컨테이너가 같은 OS 커널을 공유한다는 점이 다름.
- **이미지 vs 컨테이너**: 이미지는 가상 환경에서 작업할 패키지(템플릿)이고, 컨테이너는 그 이미지의 실행 인스턴스다.
- 도커파일로 이미지를 만들어두면 어떤 컨테이너 플랫폼에서도 동일하게 구동되므로, 운영팀은 인프라 설치 설명서 대신 이미지만으로 배포할 수 있다.

컨테이너를 패키징한 다음에는 (1) 어떻게 구동할지, (2) 다른 컨테이너·DB·백엔드와 어떻게 의존성을 맺을지, (3) 사용자가 늘 때 어떻게 확장/축소할지를 관리해야 한다 — 이 모든 자동 배치·관리 과정을 **컨테이너 오케스트레이션**이라 부르며, Kubernetes가 대표적인 구현체다.

## Master와 Worker 컴포넌트 개요

Kubernetes 클러스터는 **Master 노드**(제어 영역, control plane)와 **Worker 노드**로 구성된다.

- Master 노드는 클러스터를 관리하고, 노드 정보를 저장하고, 어떤 컨테이너가 어디로 갈지 계획하고, 노드와 컨테이너를 모니터링한다.
- 컨테이너 런타임(Docker, Container D, Rocket 등)은 컨트롤 플레인 컴포넌트를 컨테이너로 띄우려면 master를 포함한 모든 노드에 설치되어야 한다.
- Master의 핵심 컴포넌트: `kube-apiserver`, `ETCD`, `kube-scheduler`, `kube-controller-manager`.
- Worker의 핵심 컴포넌트: `kubelet`, `kube-proxy`, 컨테이너 런타임.

## ETCD

- 분산·신뢰성 있는 key-value store로, 단순하고 안전하고 빠름. 전통적인 행/열 DB가 아니라 key로 value를 저장·조회하며 중복 key는 허용하지 않음.
- 기본 포트 `2379`로 서비스를 열고, `etcdctl` 클라이언트로 값을 저장/조회한다 (`etcdctl set key1 value1`, `etcdctl get key1`).
- Kubernetes에서 ETCD는 노드, 파드, 설정, 시크릿, 계정, 역할, 바인딩 등 클러스터의 모든 상태 정보를 저장하는 단일 진실 공급원(source of truth)이다. `kubectl get`으로 보이는 모든 정보, `kubectl` 로 만든 모든 변경 사항은 ETCD에 반영되어야 비로소 "완료"로 간주된다.
- 클러스터를 kubeadm으로 구성하면 ETCD는 `kube-system` 네임스페이스의 파드로 배치된다. 스스로 구성하면 바이너리를 직접 설치해야 한다.
- 고가용성 환경에서는 여러 마스터 노드에 걸쳐 여러 ETCD 인스턴스가 있으며, `initial-cluster` 옵션으로 인스턴스끼리 서로를 알게 해야 한다.
- `etcdctl`은 API 버전 2/3을 지원하며(`ETCDCTL_API=3` 환경변수로 버전 지정), 인증서 경로(`--cacert`, `--cert`, `--key`)를 함께 지정해야 접근할 수 있다.

## kube-apiserver

- Kubernetes의 1차 관리 컴포넌트로, 클러스터 내 모든 오케스트레이션 작업을 담당한다. `kubectl` 명령이나 직접 REST 호출 모두 결국 `kube-apiserver`로 향한다.
- 요청을 인증·검증한 뒤 ETCD에서 데이터를 읽거나 쓴다. **ETCD와 직접 통신하는 유일한 컴포넌트**이며, 스케줄러·컨트롤러 매니저·kubelet은 모두 API 서버를 통해 클러스터를 갱신한다.
- 파드 생성 흐름 예시: API 서버가 노드 미배정 상태로 파드 오브젝트를 만들고 ETCD에 기록 → 스케줄러가 적합한 노드를 찾아 API 서버에 통지 → API 서버가 ETCD를 갱신하고 해당 노드의 kubelet에 전달 → kubelet이 컨테이너 런타임에 이미지를 배포하도록 지시 → 완료되면 kubelet이 상태를 API 서버에 보고하고 ETCD가 다시 갱신됨.
- kubeadm 클러스터에서는 `kube-system` 네임스페이스의 파드로 배치되며, 옵션은 해당 파드 정의 파일에서 확인 가능. 수동 구성 클러스터에서는 서비스 파일이나 실행 중인 프로세스에서 옵션을 확인한다.

## kube-controller-manager

- 여러 컨트롤러를 하나의 프로세스로 패키징해 관리한다. **컨트롤러**란 시스템의 여러 컴포넌트 상태를 지속적으로 모니터링하고, 전체 시스템을 원하는 상태로 되돌리는 프로세스다.
- **Node Controller**: 5초마다 노드 상태를 점검. 하트비트가 끊기면 40초 후 `unreachable`로 표시하고, 5분간 더 기다린 뒤에도 복구되지 않으면 해당 노드의 파드를(레플리카셋 소속이면) 다른 정상 노드로 재배치한다.
- **Replication Controller**: 레플리카셋의 상태를 모니터링해 항상 지정된 개수의 파드가 떠 있도록 보장. 파드가 죽으면 새로 만든다.
- Deployment, Service, Namespace, PersistentVolume 등 지금까지 배운 대부분의 지능적인 동작이 이 다양한 컨트롤러들을 통해 구현된다.
- `--controllers` 옵션으로 활성화할 컨트롤러를 선택할 수 있다(기본은 전부 활성화). kubeadm 클러스터에서는 `kube-system` 네임스페이스의 파드로 배치된다.

## kube-scheduler

- 어떤 파드를 어떤 노드에 배치할지 **결정만** 하는 컴포넌트 — 실제로 파드를 노드에 생성하는 것은 `kubelet`의 역할이다.
- 두 단계로 동작:
  1. **필터링**: 파드의 CPU/메모리 요구사항을 충족하지 못하는 노드를 제외.
  2. **순위 매기기**: 남은 노드에 우선순위 함수로 0~10점을 매겨(예: 파드 배치 후 남는 자원량) 가장 적합한 노드를 선택.
- 커스터마이징이 가능하며 자체 스케줄러를 작성할 수도 있음(세부 내용은 [스케줄링](../cka-with-practice-tests/scheduling.md) 참고).
- kubeadm 클러스터에서는 `kube-system`의 파드로 배치된다.

## kubelet

- Worker 노드에서 동작하는 에이전트로, 노드를 클러스터에 등록하고 `kube-apiserver`의 지시를 받아 컨테이너 런타임에 이미지 pull·인스턴스 생성을 요청한다.
- 파드·컨테이너 상태를 지속적으로 모니터링해 `kube-apiserver`에 주기적으로 보고한다.
- **kubeadm은 kubelet을 자동으로 설치하지 않는다** — 다른 컨트롤 플레인 컴포넌트와 달리 워커 노드에 수동으로 설치해야 한다.

## kube-proxy

- 클러스터 내 모든 파드가 서로 통신할 수 있게 하는 파드 네트워크(가상 네트워크)가 있지만, 파드 IP는 재시작 시 바뀔 수 있어 `Service`를 통해 접근하는 것이 안전하다.
- Service는 실제 컨테이너/리스닝 프로세스가 없는 가상 컴포넌트로, 파드 네트워크에 직접 참여하지 않는다. **kube-proxy**가 각 노드에서 동작하며 새 서비스가 생성될 때마다 해당 서비스로의 트래픽을 백엔드 파드로 전달하는 규칙(주로 iptables 규칙)을 각 노드에 생성한다.
- kubeadm은 kube-proxy를 각 노드에 **DaemonSet**으로 배포한다(DaemonSet 개념은 [스케줄링](../cka-with-practice-tests/scheduling.md) 참고).

## kubectl

- `kube-apiserver`와 통신하는 커맨드라인 도구.
```bash
kubectl run hello-minikube   # 클러스터에 애플리케이션 배치
kubectl cluster-info         # 클러스터 정보 조회
kubectl get nodes            # 클러스터 노드 목록 조회
```

## Namespace

- 같은 물리 클러스터를 여러 팀/환경(예: dev, prod)으로 논리적으로 나눌 때 사용.
- 기본적으로 `default`, `kube-system`(컨트롤 플레인 컴포넌트), `kube-public` 네임스페이스가 존재.
- 다른 네임스페이스의 오브젝트를 참조하려면 `<서비스명>.<네임스페이스>.svc.cluster.local` 형식을 사용.
```bash
kubectl create namespace dev
kubectl get pods --namespace=dev
kubectl config set-context --current --namespace=dev   # 기본 네임스페이스 전환
```
- ResourceQuota 오브젝트로 네임스페이스별 리소스 총량을 제한할 수 있다.

## Imperative vs Declarative

- **Imperative(명령형)**: `kubectl run`, `kubectl create`, `kubectl edit`, `kubectl scale`처럼 "어떻게" 할지 매 단계를 직접 지시하는 방식. 빠르지만 이력 관리가 어려움.
- **Declarative(선언형)**: YAML 정의 파일에 "원하는 최종 상태"를 기술하고 `kubectl apply -f`로 적용하는 방식. Kubernetes가 현재 상태와 원하는 상태의 차이를 계산해 반영한다. 실무·시험 모두에서 권장되는 방식.
- 시험/실무 팁: 오브젝트를 빠르게 만들고 싶을 때는 `kubectl run`/`kubectl create --dry-run=client -o yaml`로 스캐폴드를 뽑은 뒤 편집해서 `apply`하는 하이브리드 방식이 실용적이다.

# 색인과 출처
- Kubernetes for the Absolute Beginners (KodeKloud/Udemy) — Container Orchestration, Kubernetes Architecture 섹션
- Certified Kubernetes Administrator with Practice Tests (KodeKloud/Udemy) — Core Concepts 섹션 (Cluster Architecture, ETCD, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, kube-proxy, Namespaces, Imperative vs Declarative)
