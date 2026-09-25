# 로깅과 모니터링

## 목차
- [클러스터 모니터링](#클러스터-모니터링)
- [Metrics Server 설치와 사용](#metrics-server-설치와-사용)
- [애플리케이션 로그 조회](#애플리케이션-로그-조회)

---

## 클러스터 모니터링

Kubernetes는 자체적으로 완전한 기능의 모니터링 솔루션을 내장하고 있지 않다. 필요한 지표(노드 수/상태, CPU·메모리·네트워크·디스크 사용량, 파드 수와 파드별 리소스 사용량 등)를 얻으려면 별도 솔루션이 필요하다.

- **오픈소스**: Metrics Server, Prometheus, Elastic Stack 등
- **상용**: Datadog, Dynatrace 등
- Heapster는 초기 프로젝트였으나 현재는 Deprecated — 경량화된 후속 프로젝트가 **Metrics Server**다.

Metrics Server는 클러스터당 1개만 둘 수 있고, 각 노드/파드에서 지표를 모아 **메모리에만** 집계한다 — 디스크에 저장하지 않으므로 히스토리컬(과거) 데이터는 볼 수 없다. 과거 추세를 보려면 Prometheus 등 고급 솔루션이 필요하다.

### 지표가 만들어지는 경로

`kubelet`에는 `cAdvisor`(Container Advisor)라는 서브컴포넌트가 있어 파드의 성능 지표를 수집하고, `kubelet` API를 통해 Metrics Server가 가져갈 수 있도록 노출한다.

## Metrics Server 설치와 사용

```bash
# Minikube
minikube addons enable metrics-server

# 그 외 환경 (예: kodekloud 배포 매니페스트 사용)
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
kubectl create -f kubernetes-metrics-server/
```

배포 후 지표가 수집되기까지 몇 분 정도 걸린다.

```bash
kubectl top node                              # 노드별 CPU/메모리 사용량
kubectl top node --sort-by='cpu' --no-headers | head -1     # CPU 최다 사용 노드
kubectl top pod                               # 파드별 CPU/메모리 사용량
kubectl top pod --sort-by='memory' --no-headers | head -1   # 메모리 최다 사용 파드
```

## 애플리케이션 로그 조회

Docker에서는 `docker logs <container-id>`(`-f`로 실시간 스트림)로 로그를 본다. Kubernetes에서도 동일한 개념이다.

```bash
kubectl logs <pod-name>              # 컨테이너가 하나뿐인 파드
kubectl logs -f <pod-name>           # 실시간 스트림
kubectl logs <pod-name> <container-name>   # 컨테이너가 여러 개인 파드는 이름을 반드시 지정
```

- 파드에 컨테이너가 여러 개인데 컨테이너 이름을 지정하지 않으면, `kubectl logs`는 어떤 컨테이너를 보여줄지 알 수 없어 에러를 낸다.
- 애플리케이션 개발자 입장에서 필요한 로깅 지식은 사실상 이 정도가 전부다. 로그 수집기를 클러스터 전체에 통합하는 고급 구성(Fluentd/Elastic Stack 연동 등)은 별도 주제로 다룬다.

# 색인과 출처
- Certified Kubernetes Administrator with Practice Tests (KodeKloud/Udemy) — Logging and Monitoring 섹션 (Monitor Cluster Components, Managing Application Logs)
