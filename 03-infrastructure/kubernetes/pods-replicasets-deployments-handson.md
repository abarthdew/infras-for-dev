# Pod / ReplicaSet / Deployment 실습 정리

## 목차
- [Pod 정의 파일 구조](#pod-정의-파일-구조)
- [Pod 생성/조회 명령](#pod-생성조회-명령)
- [ReplicationController와 ReplicaSet](#replicationcontroller와-replicaset)
- [ReplicaSet 스케일링](#replicaset-스케일링)
- [Deployment](#deployment)
- [Deployment 업데이트와 롤백](#deployment-업데이트와-롤백)
- [POD/Deployment 편집 시 주의사항](#poddeployment-편집-시-주의사항)

---

## Pod 정의 파일 구조

Kubernetes 정의 파일은 항상 네 개의 최상위(root-level) 필드를 가진다 — `apiVersion`, `kind`, `metadata`, `spec`. 전부 필수 필드다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx
```

- `apiVersion`: 만들려는 오브젝트에 맞는 API 버전(Pod는 `v1`, Deployment/ReplicaSet은 `apps/v1`).
- `kind`: 오브젝트 종류(`Pod`, `ReplicaSet`, `Deployment`, `Service` 등).
- `metadata`: `name`, `labels` 등 오브젝트에 대한 메타 정보(딕셔너리). `labels` 아래에는 임의의 key-value를 자유롭게 추가할 수 있지만, `metadata` 바로 아래에는 Kubernetes가 허용하는 필드만 올 수 있다.
- `spec`: 오브젝트별로 다른 상세 스펙. Pod의 경우 `containers`가 배열이며, 파드 하나에 여러 컨테이너를 둘 수 있다.
- YAML 들여쓰기는 공백 2칸을 권장하며 탭은 피한다. 형제 속성은 반드시 같은 들여쓰기 레벨을 가져야 한다.

## Pod 생성/조회 명령

```bash
kubectl create -f pod-definition.yml     # 또는 kubectl apply -f
kubectl get pods
kubectl describe pod myapp-pod           # 컨테이너, 라벨, 이벤트 등 상세 정보
```

- `kubectl run <name> --image=<image>`는 내부적으로 파드를 자동 생성한다. 이미지는 Docker Hub 등 레지스트리에서 pull된다.

## ReplicationController와 ReplicaSet

레플리카를 두는 이유는 (1) 파드가 죽었을 때 자동 복구를 통한 고가용성, (2) 여러 파드로 부하를 분산하기 위함이다. **ReplicationController**는 구형 기술이고 **ReplicaSet**이 새 표준이다.

```yaml
# replicaset-definition.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  template:                 # Pod 정의를 그대로 옮겨 넣음 (apiVersion/kind 제외)
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx
          image: nginx
  selector:
    matchLabels:
      app: myapp
```

- `apiVersion`은 반드시 `apps/v1`이어야 한다. `v1`으로 쓰면 `no matches for kind "ReplicaSet"` 오류가 난다.
- ReplicaSet은 **selector가 필수**다 — ReplicationController와 달리 라벨이 일치하기만 하면 ReplicaSet 생성 이전에 만들어진 기존 파드도 관리 대상에 포함시킬 수 있기 때문이다.
- `template` 섹션은 파드가 하나도 없어도 필요하다 — 나중에 파드가 죽었을 때 새로 만들 때 사용되기 때문.

```bash
kubectl create -f replicaset-definition.yml
kubectl get replicaset
kubectl get pods
kubectl describe replicaset myapp-replicaset
kubectl delete replicaset myapp-replicaset   # 관리 중인 파드도 함께 삭제됨
```

## ReplicaSet 스케일링

```bash
# 방법 1: 정의 파일의 replicas 값을 수정한 뒤
kubectl replace -f replicaset-definition.yml

# 방법 2: 커맨드로 바로 조정 (정의 파일은 자동으로 갱신되지 않음에 주의)
kubectl scale --replicas=6 -f replicaset-definition.yml
kubectl scale --replicas=6 replicaset myapp-replicaset
```

## Deployment

Deployment는 ReplicaSet보다 상위에 있는 오브젝트로, **롤링 업데이트, 롤백, 일시정지/재개(pause/resume)** 기능을 제공한다. 정의 파일 구조는 `kind: Deployment`만 다를 뿐 ReplicaSet과 거의 동일하다.

```bash
kubectl create -f deployment-definition.yml
kubectl get deployments
kubectl get replicaset   # Deployment가 자동으로 만든 ReplicaSet 확인
kubectl get pods
kubectl get all
```

- Deployment 생성 시 내부적으로 ReplicaSet을, ReplicaSet은 다시 Pod를 만드는 계층 구조다.

## Deployment 업데이트와 롤백

- 배포 전략은 두 가지:
  - **Recreate**: 기존 인스턴스를 모두 내린 뒤 새 버전을 올림 — 다운타임 발생, 기본 전략이 아님.
  - **RollingUpdate**(기본값): 이전 버전을 하나씩 내리면서 새 버전을 하나씩 올려 무중단 업그레이드.
- 업그레이드마다 새로운 리비전이 생성되어 이력이 남는다.

```bash
kubectl apply -f deployment-definition.yml           # 정의 파일 수정 후 반영
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1   # 이미지만 바로 교체

kubectl rollout status deployment/myapp-deployment    # 롤아웃 진행 상태
kubectl rollout history deployment/myapp-deployment    # 리비전 이력

kubectl rollout undo deployment/myapp-deployment       # 직전 리비전으로 롤백
```

- 내부적으로 업그레이드 시 새 ReplicaSet을 만들고 그쪽으로 파드를 점진적으로 옮기며, 롤백 시에는 반대로 새 ReplicaSet을 0개로 줄이고 이전 ReplicaSet을 다시 늘린다.
- `kubectl set image`로 직접 갱신하면 정의 파일 자체는 갱신되지 않으므로, 이후 같은 파일로 다시 apply할 때 값이 어긋날 수 있어 주의가 필요하다.

## POD/Deployment 편집 시 주의사항

이미 생성된 **Pod**는 다음 필드 외에는 수정할 수 없다.
- `spec.containers[*].image`
- `spec.initContainers[*].image`
- `spec.activeDeadlineSeconds`
- `spec.tolerations`

다른 필드(환경 변수, 서비스 계정, 리소스 제한 등)를 바꾸려면:
1. `kubectl edit pod <name>` → 편집 시도는 거부되지만, 변경 내용이 담긴 임시 파일 경로가 안내됨 → 기존 파드를 삭제하고 그 임시 파일로 재생성.
2. 또는 `kubectl get pod <name> -o yaml > my-new-pod.yaml`로 내보낸 뒤 직접 편집 → 기존 파드 삭제 → 새 파일로 재생성.

**Deployment**는 파드 템플릿이 Deployment 스펙의 자식이므로 `kubectl edit deployment <name>`으로 어떤 필드든 바로 편집할 수 있다 — 변경 시 자동으로 새 파드가 만들어진다.

# 색인과 출처
- Kubernetes for the Absolute Beginners (KodeKloud/Udemy) — PODs with YAML, ReplicaSets, Deployments 섹션
- Certified Kubernetes Administrator with Practice Tests (KodeKloud/Udemy) — Core Concepts 섹션 (Recap PODs/ReplicaSets/Deployments), A quick note on editing PODs and Deployments
