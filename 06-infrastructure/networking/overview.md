# Infrastructure Networking

## 목차
- VPC와 subnet
- routing table
- security group
- firewall
- load balancer
- private/public network

## 기초 개념
인프라 네트워킹은 서버들이 어떤 네트워크에 있고, 어떤 경로와 규칙으로 서로 통신할 수 있는지 설계하는 영역이다.

```text
internet -> load balancer -> public subnet -> private subnet
```

## 간단한 예시
```text
public subnet   load balancer, bastion
private subnet  app server, database
```

## 반드시 알아야 할 질문
- public subnet과 private subnet은 무엇이 다른가?
- security group과 firewall은 어떤 역할을 하는가?
- NAT gateway는 왜 필요한가?
- load balancer는 어느 계층에서 동작하는가?
