# Service 실습 보충 — NodePort의 3-포트 모델과 예제

> Service의 기본 개념(ClusterIP/NodePort/LoadBalancer 타입, 기본 YAML 구조)은 [overview.md](./overview.md#service)에 정리되어 있다. 이 문서는 NodePort 동작 원리와 실습에서 자주 막히는 포인트를 보충한다.

## 목차
- [NodePort의 3개 포트](#nodeport의-3개-포트)
- [NodePort YAML 예제](#nodeport-yaml-예제)
- [여러 Pod / 여러 Node로 확장될 때](#여러-pod--여러-node로-확장될-때)
- [ClusterIP 예제](#clusterip-예제)
- [LoadBalancer 타입의 실제 동작](#loadbalancer-타입의-실제-동작)

---

## NodePort의 3개 포트

NodePort 서비스에는 관점이 다른 세 개의 포트가 있다.

1. **targetPort**: 실제 웹 서버가 떠 있는 파드 내부 포트 (예: `80`).
2. **port**: Service 자체의 포트. Service는 클러스터 내부의 가상 서버처럼 동작하며 자신만의 ClusterIP를 가진다.
3. **nodePort**: 외부에서 노드 IP로 접근할 때 쓰는 포트. 기본 유효 범위는 `30000~32767`.

세 필드 중 **`port`만 필수**다. `targetPort`를 생략하면 `port`와 같은 값으로 간주되고, `nodePort`를 생략하면 유효 범위 내에서 자동 할당된다.

## NodePort YAML 예제

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:        # Service를 Pod와 연결하는 필수 항목
    app: myapp      # pod-definition의 labels와 동일해야 함
```

```bash
kubectl create -f service-definition.yml
kubectl get services
curl http://<노드IP>:30008
```

- Service 정의 파일 자체에는 어떤 파드에 연결할지 명시하는 필드가 없어 보이지만, `selector`가 그 역할을 한다 — ReplicaSet/Deployment와 마찬가지로 라벨을 매칭해 대상을 찾는다.

## 여러 Pod / 여러 Node로 확장될 때

- 동일 라벨을 가진 파드가 여러 개면, Service는 추가 설정 없이 자동으로 모든 매칭 파드를 엔드포인트로 등록하고 **랜덤 알고리즘**으로 부하를 분산한다.
- 파드가 여러 노드에 흩어져 있어도, Service는 별도 설정 없이 **모든 노드에서 동일한 nodePort로 접근 가능**하도록 자동 구성된다 — 단일 파드/단일 노드든, 다중 파드/다중 노드든 Service 생성 과정 자체는 동일하다.
- 파드가 추가/제거되면 Service가 자동으로 갱신된다.

## ClusterIP 예제

여러 계층(frontend/backend/redis/db)으로 구성된 애플리케이션에서, 파드 IP는 재시작 시 바뀌므로 내부 통신에 신뢰할 수 없다. 각 계층마다 ClusterIP 서비스를 두면, 서비스 이름 자체가 클러스터 내부 DNS처럼 동작해 안정적으로 참조할 수 있다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  # type: ClusterIP   # 기본값이므로 생략 가능
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    tier: backend
```

## LoadBalancer 타입의 실제 동작

- NodePort만으로 외부에 서비스를 노출하면, 사용자에게 "노드 IP + 포트" 조합을 여러 개 알려줘야 하는 문제가 생긴다(노드 수만큼 URL 발생).
- 자체적으로 로드밸런서 VM을 구성해 NodePort들 앞단에 두는 방법도 있지만, GCP/AWS/Azure 같은 지원되는 클라우드 플랫폼에서는 `type: LoadBalancer`만 지정하면 해당 클라우드의 네이티브 로드밸런서가 자동으로 프로비저닝된다.
- **주의**: 지원되지 않는 환경(VirtualBox 등 로컬 환경)에서 `LoadBalancer` 타입을 쓰면 `NodePort`와 동일하게 동작할 뿐, 외부 로드밸런서 구성은 이뤄지지 않는다.

# 색인과 출처
- Kubernetes for the Absolute Beginners (KodeKloud/Udemy) — Services (NodePort/ClusterIP/LoadBalancer) 섹션
- Certified Kubernetes Administrator with Practice Tests (KodeKloud/Udemy) — Core Concepts 섹션 (Services 실습 보충)
