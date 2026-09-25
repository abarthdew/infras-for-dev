# 클러스터 유지보수: 노드 점검과 버전 업그레이드

## 목차
- [노드 점검: drain / cordon / uncordon](#노드-점검-drain--cordon--uncordon)
- [Kubernetes 버전 체계](#kubernetes-버전-체계)
- [클러스터 업그레이드 프로세스](#클러스터-업그레이드-프로세스)

---

## 노드 점검: drain / cordon / uncordon

노드를 유지보수(OS 패치, 재부팅 등)를 위해 내려야 할 때 Kubernetes가 어떻게 동작하는지 이해하는 게 먼저다.

- 노드가 내려가도 바로 돌아오면(짧은 다운타임) `kubelet`이 다시 뜨면서 그 노드의 Pod들도 그대로 돌아온다.
- 노드가 **5분 이상** 내려가 있으면, Kubernetes는 그 노드를 죽은 것으로 간주하고 노드에 있던 Pod들을 종료 처리한다.
  - 이 5분 값이 **Pod eviction timeout**이며, Controller Manager에 설정된 값(기본 5분)이다.
  - ReplicaSet에 속한 Pod라면 다른 노드에 새로 생성된다.
  - ReplicaSet 없이 단독으로 떠 있던 Pod라면 그냥 사라진다(복구되지 않음).

즉, 노드가 5분 안에 확실히 돌아온다는 보장이 없다면(대부분 그렇다), 무작정 재부팅하기보다 **의도적으로 노드를 비우는** 절차를 쓰는 게 안전하다.

```bash
kubectl drain node-1
```

- `drain`: 노드 위 Pod들을 **정상적으로(gracefully) 종료**시키고 다른 노드에 재생성한다. 동시에 노드를 **cordon**(스케줄링 불가 상태)으로 표시해, 점검이 끝날 때까지 새 Pod가 배치되지 않게 한다.

```bash
kubectl uncordon node-1
```

- `uncordon`: 점검이 끝난 노드를 다시 스케줄링 가능 상태로 되돌린다. 단, drain으로 옮겨졌던 Pod가 자동으로 이 노드로 돌아오지는 않는다 — 이후 새로 생성되는 Pod나 삭제 후 재생성되는 Pod부터 이 노드에 배치될 수 있다.

```bash
kubectl cordon node-1
```

- `cordon`: drain과 달리 기존 Pod를 건드리거나 옮기지 않고, **새 Pod만 스케줄링되지 않도록** 막는다.

## Kubernetes 버전 체계

버전은 `메이저.마이너.패치` 3단으로 구성된다 (예: `1.13.0`). 마이너 버전은 몇 달 주기로 새 기능과 함께 릴리스되고, 패치는 더 자주 나오는 버그 수정 릴리스다.

- 릴리스 단계: **alpha**(기본 비활성화, 버그 가능성 있음) → **beta**(코드가 충분히 테스트됨, 기본 활성화) → **stable**.
- Kubernetes는 항상 **최근 3개 마이너 버전만 지원**한다. 예를 들어 최신이 1.12라면 지원 범위는 1.12/1.11/1.10.
- ETCD, CoreDNS 등 컨트롤 플레인 안의 외부 의존 컴포넌트는 Kubernetes 자체와 별도의 버전 체계를 갖는다.

## 클러스터 업그레이드 프로세스

### 컴포넌트 간 버전 스큐(skew) 규칙

- kube-apiserver가 전체 컨트롤 플레인의 기준선이며, 다른 어떤 컴포넌트도 apiserver보다 **높은** 버전일 수 없다.
- controller-manager/scheduler는 apiserver보다 최대 1버전 낮을 수 있다(`X-1`).
- kubelet/kube-proxy는 apiserver보다 최대 2버전 낮을 수 있다(`X-2`).
- `kubectl`만 예외적으로 apiserver보다 1버전 높거나(`X+1`), 같거나, 낮아도(`X-1`) 된다.
- 이 허용 스큐 덕분에 전체를 한 번에 내리지 않고 컴포넌트별로 순차적인 라이브 업그레이드가 가능하다.

### 업그레이드 방법

- 관리형 클라우드 Kubernetes(GKE 등)는 몇 번의 클릭으로 업그레이드 가능.
- `kubeadm`으로 구성한 클러스터는 `kubeadm`이 계획/실행을 도와줌.
- 직접 처음부터 구성한 클러스터는 컴포넌트를 수동으로 하나씩 업그레이드.
- **마이너 버전은 한 번에 하나씩만** 올린다 (1.10 → 1.13으로 바로 가지 않고 1.10→1.11→1.12→1.13 순).

### kubeadm 기반 업그레이드 절차

1. **마스터 노드 업그레이드**
   ```bash
   kubeadm upgrade plan      # 현재 버전, kubeadm 버전, 업그레이드 가능한 버전 확인
   apt-get upgrade -y kubeadm=1.12.x-00
   kubeadm upgrade apply v1.12.0
   apt-get upgrade -y kubelet=1.12.x-00
   systemctl restart kubelet
   ```
   - 마스터가 업그레이드되는 동안 API 서버/스케줄러/컨트롤러 매니저가 잠깐 내려가지만, 이미 떠 있는 워커 노드의 애플리케이션은 계속 서비스된다. 다만 `kubectl`로 클러스터를 조작하거나 배포/롤백을 할 수 없고, Pod가 죽어도 자동 재생성되지 않는다(컨트롤러 매니저가 멈춰 있으므로).
   - `kubeadm upgrade apply`는 한 번에 한 마이너 버전씩만 가능 — 여러 버전을 건너뛰려면 `kubeadm` 자체를 먼저 목표 버전으로 올린 뒤 반복.

2. **워커 노드 업그레이드** (전략 3가지)
   - **한꺼번에 전부 업그레이드**: 가장 간단하지만 업그레이드 동안 전체 다운타임 발생.
   - **한 노드씩 순차 업그레이드**: 다운타임 없이 안전하지만 더 오래 걸림.
     ```bash
     kubectl drain node-1
     apt-get upgrade -y kubeadm=1.12.x-00 kubelet=1.12.x-00
     kubeadm upgrade node
     systemctl restart kubelet
     kubectl uncordon node-1
     # 다음 노드도 동일하게 반복
     ```
   - **새 버전 노드를 추가하고 구 버전 노드를 제거**: 클라우드 환경에서 노드 프로비저닝이 쉬울 때 특히 유용. 워크로드를 새 노드로 옮긴 뒤 구 노드를 제거.
   - `kubectl get nodes`에 표시되는 버전은 각 노드의 kubelet 버전이지 apiserver 버전이 아니라는 점에 주의 — 마스터를 업그레이드해도 워커 노드 라인은 그대로 이전 버전으로 보인다.

# 색인과 출처
- Certified Kubernetes Administrator with Practice Tests (KodeKloud/Udemy) — Cluster Maintenance 섹션 (OS Upgrades, Kubernetes Software Versions, Cluster Upgrade Process)
