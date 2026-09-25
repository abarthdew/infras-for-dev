# 스케줄링 심화 — 수동 스케줄링부터 리소스 제한까지

## 목차
- [수동 스케줄링](#수동-스케줄링)
- [Labels와 Selectors](#labels와-selectors)
- [Taints와 Tolerations](#taints와-tolerations)
- [Node Selector와 Node Affinity](#node-selector와-node-affinity)
- [Taints/Tolerations vs Node Affinity](#taintstolerations-vs-node-affinity)
- [리소스 요청과 제한](#리소스-요청과-제한)
- [DaemonSet](#daemonset)
- [Static Pod](#static-pod)

---

## 수동 스케줄링

- 모든 Pod는 기본적으로 비어 있는 `nodeName` 필드를 가진다. 스케줄러는 이 필드가 비어 있는 파드를 찾아 스케줄링 대상으로 삼고, 알고리즘으로 적절한 노드를 찾은 뒤 `nodeName`을 채워 넣는다(binding 오브젝트를 통해).
- 스케줄러가 없는 클러스터에서는 파드가 계속 `Pending` 상태로 남는다.
- 파드 생성 시점에는 `spec.nodeName`을 직접 지정해 수동으로 스케줄링할 수 있다. 단, **이미 생성된 파드의 `nodeName`은 수정할 수 없다** — 이 경우 binding 오브젝트를 만들어 Pod Binding API로 POST 요청을 보내야 한다(스케줄러가 실제로 하는 일을 그대로 흉내 내는 방식).

## Labels와 Selectors

- 라벨과 셀렉터는 Kubernetes 오브젝트를 그룹화하고 필터링하는 표준 수단이다. `metadata.labels`에 원하는 key-value를 자유롭게 추가하고, `kubectl get pods --selector app=App1`처럼 조회한다.
- ReplicaSet/Deployment/Service는 내부적으로 라벨과 셀렉터로 서로를 연결한다. 초보자가 자주 헷갈리는 지점: ReplicaSet 정의 파일에는 라벨이 **두 곳**에 나온다 — 최상위 `metadata.labels`는 ReplicaSet 자체의 라벨이고, `spec.template.metadata.labels`가 실제로 생성되는 Pod의 라벨이다. ReplicaSet이 Pod를 찾으려면 `spec.selector.matchLabels`를 Pod의 라벨과 일치시켜야 한다.
- **Annotation**은 라벨과 달리 그룹화/필터링용이 아니라, 툴 버전·빌드 정보·담당자 연락처 등 순수 정보 기록용으로 쓰인다.

## Taints와 Tolerations

- Taint는 **Node**에, Toleration은 **Pod**에 설정한다. Taint/Toleration은 보안이나 침입 차단과 무관하며, 어떤 Pod가 어떤 Node에 스케줄될 수 있는지를 제한하는 용도다.
- 기본적으로 Pod는 어떤 taint도 tolerate하지 않으므로, 노드에 taint를 걸면 명시적으로 toleration을 추가한 파드만 그 노드에 배치될 수 있다.

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

- Taint effect는 세 가지:
  - `NoSchedule`: 조건에 맞지 않는 파드는 스케줄되지 않음.
  - `PreferNoSchedule`: 가능하면 피하지만 보장하지는 않음.
  - `NoExecute`: 새 파드는 스케줄되지 않고, **이미 실행 중인 파드도 tolerate하지 않으면 evict(축출)됨**.

```yaml
spec:
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

- Taint/Toleration은 "이 파드를 이 노드로 보내라"가 아니라 **"이 노드는 이 toleration을 가진 파드만 받아라"**라는 제약일 뿐이다 — 다른 노드에 taint가 없다면 해당 파드가 다른 노드에 배치될 수도 있다. 특정 파드를 특정 노드로 유도하려면 Node Affinity가 필요하다.
- 클러스터를 처음 구성하면 **Master 노드에는 자동으로 `NoSchedule` taint가 걸려** 일반 워크로드가 배치되지 않는다(운영 애플리케이션을 마스터에 배치하지 않는 것이 권장 사항). `kubectl describe node <master>`의 Taints 항목에서 확인 가능.

## Node Selector와 Node Affinity

**Node Selector**(단순한 방법):
```yaml
spec:
  nodeSelector:
    size: Large
```
- 사전에 `kubectl label nodes node-1 size=Large`로 노드를 라벨링해둬야 한다.
- "large 또는 medium", "small이 아닌 것" 같은 복합 조건은 표현할 수 없다는 한계가 있다.

**Node Affinity**(고급 방법):
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In         # In, NotIn, Exists 등
                values:
                  - Large
                  - Medium
```

- Node Affinity의 타입은 **스케줄링 시점**과 **실행 중 시점**을 각각 다르게 다룬다.
  - `requiredDuringSchedulingIgnoredDuringExecution`: 조건에 맞는 노드가 없으면 파드가 아예 스케줄되지 않음(배치가 필수적인 경우).
  - `preferredDuringSchedulingIgnoredDuringExecution`: 조건에 맞는 노드가 없으면 아무 노드에나 배치(배치보다 워크로드 실행 자체가 중요한 경우).
  - 두 타입 모두 `...IgnoredDuringExecution`이므로, 이미 스케줄된 뒤 노드의 라벨이 바뀌어도 실행 중인 파드에는 영향이 없다. (향후 `requiredDuringExecution` 타입이 추가되면, 라벨이 바뀔 경우 조건에 맞지 않는 파드를 evict하게 될 예정이었음.)

## Taints/Tolerations vs Node Affinity

- Taint/Toleration만으로는 "다른 팀의 파드가 내 노드에 배치되지 않게" 막을 수는 있지만, "내 파드가 다른 노드에 배치되지 않게"는 보장하지 못한다.
- Node Affinity만으로는 반대로 "내 파드를 내 노드로 유도"는 되지만, "다른 파드가 내 노드에 오지 못하게"는 막지 못한다.
- 따라서 **특정 노드를 특정 워크로드 전용으로 완전히 격리**하려면 두 개념을 함께 사용한다 — Taint/Toleration으로 다른 파드의 유입을 막고, Node Affinity로 내 파드가 다른 노드로 가지 않게 고정한다.

## 리소스 요청과 제한

- 컨테이너의 기본 리소스 요청(request)은 CPU 0.5, 메모리 256Mi로 간주된다 *(단, 이 기본값이 실제로 적용되려면 해당 네임스페이스에 `LimitRange` 오브젝트가 먼저 정의되어 있어야 한다)*. 스케줄러는 이 값을 기준으로 배치 가능한 노드를 찾는다.
- 리소스가 부족한 노드에는 배치하지 않고, 클러스터 전체에 여유가 없으면 파드는 `Pending` 상태로 남는다(`describe` 이벤트에 `Insufficient cpu` 등으로 표시).

```yaml
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      resources:
        requests:
          memory: "1Gi"
          cpu: 1
        limits:
          memory: "2Gi"
          cpu: 2
```

- CPU 단위: `1` = 1 vCPU(AWS 1vCPU / GCP·Azure 1 core / 1 Hyperthread와 동일). `0.1` CPU는 `100m`(밀리)로도 표기 가능하며, `1m`보다 작게는 지정할 수 없다.
- 메모리 단위: `G`(기가바이트, 1000MB 기준)와 `Gi`(기비바이트, 1024MiB 기준)는 다르다. `M`/`Mi`, `K`/`Ki`도 마찬가지.
- Docker 컨테이너는 기본적으로 리소스 제한이 없어 노드 자원을 소진할 수 있지만, Kubernetes는 기본 limit으로 CPU 1개, 메모리 512Mi를 적용한다.
- **CPU 제한 초과 시**: 스로틀링되어 그 이상 사용하지 못함.
- **메모리 제한 초과 시**: (CPU와 달리) 일시적으로 limit을 넘는 사용은 허용되지만, 지속적으로 초과하면 파드가 **OOMKilled**로 종료된다.

## DaemonSet

- ReplicaSet처럼 여러 파드 복제본을 관리하지만, **클러스터의 모든 노드에 정확히 1개씩** 파드를 배치한다. 노드가 추가되면 자동으로 복제본이 생기고, 노드가 제거되면 자동으로 사라진다.
- 대표적인 용도: 모든 노드에 모니터링 에이전트나 로그 수집기를 배포. `kube-proxy`도 실제로 DaemonSet으로 배포된다. Weave-net 같은 네트워킹 솔루션의 에이전트도 마찬가지.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
        - name: monitoring-agent
          image: monitoring-agent
```

```bash
kubectl create -f daemon-set-definition.yml
kubectl get daemonsets
kubectl describe daemonsets <name>
```

- Kubernetes 1.12부터 DaemonSet은 내부적으로 기본 스케줄러와 Node Affinity 규칙을 사용해 노드마다 파드를 배치한다(그 이전에는 각 파드에 `nodeName`을 직접 설정하는 방식이었음).

## Static Pod

- `kube-apiserver` 없이(즉 클러스터에 속하지 않은 독립 노드에서도) `kubelet`은 파드를 만들 수 있다 — 지정된 디렉터리(`--pod-manifest-path` 옵션, 또는 `--config`로 지정한 설정 파일 내 static pod path)를 주기적으로 스캔해, 그 안의 정의 파일로 파드를 만들고 유지·재시작·삭제까지 담당한다. 이렇게 API 서버 개입 없이 kubelet이 직접 만드는 파드를 **Static Pod**라 부른다.
- Static Pod는 **Pod만** 만들 수 있다 — ReplicaSet, Deployment, Service 등은 컨트롤 플레인 컴포넌트가 필요하므로 이 방식으로 만들 수 없다.
- 클러스터에 속한 노드라면, kubelet이 Static Pod를 만들 때 `kube-apiserver`에도 읽기 전용 미러 오브젝트를 생성한다 — `kubectl get pods`에도 보이지만 API 서버를 통해 직접 수정·삭제할 수는 없고, 반드시 노드의 manifest 폴더에서 파일을 수정/삭제해야 한다. 파드 이름 뒤에 노드 이름이 자동으로 붙는다.
- **활용**: 컨트롤 플레인 컴포넌트 자체를 Static Pod로 배포할 수 있다 — 각 마스터 노드에 kubelet만 설치한 뒤, API 서버/컨트롤러/ETCD 등의 이미지를 가리키는 정의 파일을 manifest 폴더에 두면 kubelet이 알아서 배포·재시작까지 관리한다. **kubeadm이 클러스터를 구성하는 방식이 바로 이것**이며, 그래서 `kube-system` 네임스페이스에서 컨트롤 플레인 컴포넌트들이 파드로 보이는 것이다.
- Static Pod와 DaemonSet의 차이: DaemonSet은 `kube-apiserver`를 통해 DaemonSet 컨트롤러가 관리하지만, Static Pod는 `kube-apiserver`나 다른 컨트롤 플레인 컴포넌트 없이 kubelet이 단독으로 관리한다. 둘 다 `kube-scheduler`의 영향을 받지 않는다는 공통점이 있다.

# 색인과 출처
- Certified Kubernetes Administrator with Practice Tests (KodeKloud/Udemy) — Scheduling 섹션 (Manual Scheduling, Labels and Selectors, Taints and Tolerations, Node Selectors, Node Affinity, Resource Requirements and Limits, DaemonSets, Static Pods)
