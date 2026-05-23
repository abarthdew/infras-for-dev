# Kubernetes

## 목차
- cluster와 node
- pod
- deployment
- service
- ingress
- configmap과 secret

## 기초 개념
Kubernetes는 컨테이너를 여러 서버에 배치하고, 장애 복구와 확장을 관리하는 오케스트레이션 시스템이다.

```text
desired state -> control plane -> nodes -> pods
```

## 간단한 예시
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
```

## 반드시 알아야 할 질문
- pod는 container와 무엇이 다른가?
- deployment는 왜 replica를 관리하는가?
- service는 pod IP 변화 문제를 어떻게 해결하는가?
- ingress는 nginx reverse proxy와 어떤 관계인가?
