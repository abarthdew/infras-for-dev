# Kubernetes

## 목차
- [Kubernetes의 역할](#kubernetes의-역할)
- [Cluster와 Node](#cluster와-node)
- [Pod](#pod)
- [Deployment](#deployment)
- [Service](#service)
- [Ingress](#ingress)
- [ConfigMap과 Secret](#configmap과-secret)

---

## Kubernetes의 역할

Kubernetes는 **컨테이너를 여러 서버에 배치하고, 자동으로 장애 복구와 확장을 관리하는 오케스트레이션 시스템**이다. Docker는 단일 호스트에서 컨테이너를 실행하지만, Kubernetes는 여러 서버(클러스터)에서 대규모로 관리한다.

```text
desired state (원하는 상태)
    ↓ (선언적 명시)
control plane
    ↓ (해석 및 실행)
nodes (여러 서버)
    ↓ (할당 및 실행)
pods (컨테이너 래퍼)
```

### Kubernetes 없이는 어려운 작업들

```text
Docker만 사용할 때:
- 서버 1에서 컨테이너 실행
- 서버 1이 장애나면? → 직접 다른 서버에서 시작해야 함
- 트래픽이 증가하면? → 수동으로 더 많은 컨테이너 실행
- 로그를 보려면? → 여러 서버에 SSH로 접속해서 확인

Kubernetes를 사용할 때:
- "3개 replica가 항상 실행되어야 한다" 선언
- 서버가 장애나도 자동으로 다른 곳에서 재시작
- 트래픽이 증가하면 자동으로 스케일링
- 중앙에서 모든 로그와 상태 확인
```

---

## Cluster와 Node

### Cluster (클러스터)

**Kubernetes Cluster는 여러 서버(Node)가 협력하여 컨테이너를 관리하는 시스템 전체**다.

```text
Kubernetes Cluster
    ├─ Control Plane (마스터)
    │  ├─ API Server (요청 처리)
    │  ├─ Scheduler (Pod 배치 결정)
    │  ├─ Controller Manager (상태 관리)
    │  └─ etcd (상태 저장소)
    │
    ├─ Node (워커 1)
    │  ├─ kubelet (Agent, Pod 실행 감독)
    │  ├─ kube-proxy (네트워킹)
    │  └─ pods...
    │
    └─ Node (워커 2)
       ├─ kubelet
       ├─ kube-proxy
       └─ pods...
```

### Node (노드)

**Node는 Cluster에 속한 물리 또는 가상 서버**다. Control Plane의 명령을 받아 Pod를 실행한다.

```text
특징:
- kubelet: 각 Node에서 실행되는 Agent
- kube-proxy: Node의 네트워킹 담당
- 여러 개의 Pod 실행 가능
- Node 장애 → Kubernetes가 자동으로 다른 Node에 Pod 재배치
```

### 예시

```bash
# Cluster에서 실행되는 Pod 확인
kubectl get pods -o wide
# NAME              STATUS  NODE
# webapp-pod-1      Running worker-1
# webapp-pod-2      Running worker-2
# database-pod-1    Running worker-1

# 어느 Node에서 실행 중인지 확인
kubectl describe node worker-1
# ... Pod 목록 ...
```

---

## Pod

### Pod란?

**"pod는 container와 무엇이 다른가?"**

Pod는 **Docker Container를 감싼 최소 단위의 Kubernetes 객체**다. Container 하나를 의미하기도 하고, 경우에 따라 여러 Container를 함께 실행할 수도 있다.

```text
Container (Docker)           Pod (Kubernetes)
──────────────────           ────────────────
독립적 프로세스              Kubernetes의 최소 단위
직접 실행 (docker run)       Cluster 내에서 관리
단일 Container 보통          1개 이상의 Container 포함
```

### Pod의 특징

```yaml
# 가장 간단한 Pod
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
```

```text
특징:
- 가장 작은 배포 단위
- IP 주소가 할당됨 (Pod마다 고유한 IP)
- Container들이 같은 네트워크 네임스페이스 공유
- 보통 1개 Container, 종종 사이드카 패턴으로 2-3개
- 임시적 (Pod가 재시작되면 IP도 변경됨)
- 직접 생성하지 않음 (Deployment, StatefulSet으로 관리)
```

### 여러 Container를 가진 Pod (Sidecar 패턴)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logging
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
  - name: logging-sidecar
    image: logging-agent:1.0
    # 로그 수집 담당
```

---

## Deployment

### Deployment란?

**"deployment는 왜 replica를 관리하는가?"**

Deployment는 **Pod의 여러 복제본을 자동으로 생성하고, 장애 시 재시작하며, 무중단 배포를 관리하는 Kubernetes 객체**다.

```text
Deployment (선언: 3개 Pod이 항상 실행되어야 함)
    ↓
ReplicaSet (실제로 3개 Pod 생성 및 감시)
    ↓
Pod 1, Pod 2, Pod 3
```

### 역할

1. **Replica 관리**: 정확히 N개의 Pod이 항상 실행되도록 보장
2. **장애 복구**: Pod 또는 Node가 죽으면 자동으로 새로운 Pod 시작
3. **확장/축소**: replicas 수를 변경하면 Pod 개수 조정
4. **롤링 업데이트**: 새 버전으로 무중단 배포

### 기본 구조

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
spec:
  replicas: 3  # 항상 3개의 Pod이 실행되길 원함
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Replica 관리 방식

```text
원하는 상태: replicas = 3

현재 상태: 2개 Pod 실행 중 (1개 Node 장애)
    ↓ (ReplicaSet이 감지)
원하는 상태와 다르다!
    ↓
새로운 Pod 1개 생성
    ↓
현재 상태: 3개 Pod 실행 중 ✓

---

수동으로 replicas를 5로 변경
    ↓
kubectl scale deployment webserver --replicas=5
    ↓
2개 Pod 추가 생성
    ↓
현재 상태: 5개 Pod 실행 중
```

### 무중단 배포 (Rolling Update)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # 추가로 1개까지 더 실행 가능
      maxUnavailable: 0  # 항상 최소 3개 유지
```

```text
이전 버전 (3개 Pod)
    ↓ 새 버전으로 배포
[새 Pod 1개 시작]
이전: 3개, 새로운: 1개 (총 4개 일시적으로 실행)
    ↓
[이전 Pod 1개 종료]
이전: 2개, 새로운: 1개
    ↓ (반복)
최종: 새로운 버전 3개 ✓

→ 사용자가 느끼는 중단 시간: 0초
```

---

## Service

### Service란?

**"service는 pod IP 변화 문제를 어떻게 해결하는가?"**

Service는 **Pod IP 주소의 빈번한 변화를 추상화하고, 안정적인 엔드포인트를 제공하는 Kubernetes 객체**다.

```text
Pod IP는 계속 변한다:
Pod 생성 → IP 할당 (10.0.1.5)
Pod 재시작 → 새로운 IP (10.0.1.10)
Pod 삭제/재생성 → 또 다른 IP (10.0.1.15)

Service는 이를 추상화:
Service IP (안정적, 변하지 않음)
    ↓
언제나 3개의 정상 Pod로 트래픽 분산
```

### 구조

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webserver
spec:
  selector:
    app: webserver  # 이 라벨을 가진 Pod에 트래픽 전달
  ports:
  - protocol: TCP
    port: 80        # Service가 받는 포트
    targetPort: 8080  # Pod의 실제 포트
  type: ClusterIP   # 내부 통신만 (기본값)
```

### Service 타입

```text
ClusterIP (기본값)
├─ Cluster 내부에서만 접근 가능
├─ DNS: webserver.default.svc.cluster.local
└─ 내부 마이크로서비스 통신에 사용

NodePort
├─ Cluster 외부에서도 접근 가능 (Node IP:Port)
├─ 범위: 30000-32767 포트
└─ 개발/테스트 환경에서 사용

LoadBalancer
├─ 클라우드 제공자의 로드밸런서와 연동
├─ 외부 IP 자동 할당
└─ 프로덕션 환경에서 웹 애플리케이션 노출
```

### 작동 원리

```text
클라이언트 요청
    ↓
Service (IP: 10.96.0.5, Port: 80)
    ↓ (kube-proxy가 라우팅)
정상 Pod 중 하나로 전달
    ├─ Pod 1 (10.0.1.5:8080)
    ├─ Pod 2 (10.0.1.10:8080)
    └─ Pod 3 (10.0.1.15:8080)
```

---

## Ingress

### Ingress란?

**"ingress는 nginx reverse proxy와 어떤 관계인가?"**

Ingress는 **HTTP/HTTPS 라우팅을 관리하는 Kubernetes 객체**로, Nginx나 다른 Ingress Controller를 통해 동작한다. 즉, Ingress는 **규칙의 정의**이고, **Nginx (또는 다른 구현체)는 그 규칙을 실행하는 역할**이다.

```text
Client (Internet)
    ↓
Ingress Controller (Nginx 또는 다른 구현체)
    ├─ HTTPS 종료
    ├─ 경로 기반 라우팅
    └─ 호스트명 기반 라우팅
    ↓
Service (ClusterIP)
    ↓
Pod
```

### Ingress와 Nginx의 관계

```text
Ingress (Kubernetes 관점)
- "api.example.com/api/* → api-service로 보내고
  api.example.com/web/* → web-service로 보내줄래"

Nginx Ingress Controller (실제 구현)
- Ingress 정의를 읽고
- 자동으로 nginx.conf를 생성 및 재로드
- HTTP/HTTPS 요청을 규칙대로 라우팅
```

### 기본 구조

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 3000
  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 9000
```

### 라우팅 규칙

```text
조건: 호스트명 기반
api.example.com → api-service
admin.example.com → admin-service
web.example.com → web-service

조건: 경로 기반
example.com/api/* → api-service
example.com/web/* → web-service
example.com/admin/* → admin-service

조건: HTTP 헤더 기반 (Annotation으로 확장)
X-User-Type: premium → premium-service
```

### Ingress Controller의 종류

```text
Nginx Ingress Controller
├─ 오픈소스, 가장 널리 사용됨
├─ 경로 기반, 호스트명 기반 라우팅
└─ SSL/TLS 자동 관리 (cert-manager)

AWS ALB (Application Load Balancer)
├─ AWS 네이티브
├─ EC2 기반 Kubernetes (EKS)와 연동
└─ AWS 관리형

GCP Cloud Load Balancer
├─ GCP 네이티브
├─ GKE (Google Kubernetes Engine)와 연동

HAProxy, Traefik 등
```

---

## ConfigMap과 Secret

### ConfigMap

**ConfigMap은 애플리케이션 설정을 Kubernetes 객체로 관리**한다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: postgres.default.svc.cluster.local
  DATABASE_PORT: "5432"
  APP_ENV: production
  log_level: INFO
```

### Pod에서 사용

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config
    # 또는 선택적으로
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log_level
```

### Secret

**Secret은 민감한 정보(암호, API 키 등)를 암호화하여 저장**한다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

```bash
# Secret 생성
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=secret123

# YAML으로 선언적 관리
kubectl apply -f secret.yaml
```

### ConfigMap과 Secret의 차이

```text
ConfigMap          Secret
─────────────      ──────────
일반 설정 값        민감 정보
평문 저장           암호화 (etcd에서)
크기: 1MB 이상      크기: 1MB
조작 용도           보안 목적
```

---

## 전체 흐름 예시

```yaml
# 1. ConfigMap으로 설정 관리
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info

---
# 2. Deployment로 Pod 관리
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: app
        image: myapp:1.0
        envFrom:
        - configMapRef:
            name: app-config
        ports:
        - containerPort: 8080

---
# 3. Service로 Pod 노출
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer

---
# 4. Ingress로 외부 접근 제어
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp
            port:
              number: 80
```

---

## 정리

| 개념 | 설명 |
|------|------|
| Cluster | 여러 Node가 협력하는 시스템 |
| Node | Cluster에 속한 서버 |
| Pod | Container를 감싼 최소 Kubernetes 단위 |
| Deployment | Pod 복제본의 자동 관리 및 롤링 업데이트 |
| Service | Pod IP 변화를 추상화한 안정적 엔드포인트 |
| Ingress | HTTP/HTTPS 라우팅 규칙 정의 |
| ConfigMap | 비민감 설정 관리 |
| Secret | 민감 정보 암호화 관리 |

---

## Related Notes

- [Docker](../docker/overview.md)
- [데이터 센터와 클라우드 인프라](../data-center-and-cloud/overview.md)
