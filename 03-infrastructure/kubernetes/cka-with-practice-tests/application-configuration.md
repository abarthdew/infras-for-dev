# 애플리케이션 설정: Command/Args, ConfigMap, Secret, Multi-Container Pod

## 목차
- [Command와 Args](#command와-args)
- [환경 변수](#환경-변수)
- [ConfigMap](#configmap)
- [Secret](#secret)
- [Multi-Container Pod](#multi-container-pod)
- [InitContainer](#initcontainer)
- [Self-Healing](#self-healing)

---

## Command와 Args

Docker의 `ENTRYPOINT`/`CMD`와 Pod 정의 파일의 `command`/`args`는 1:1로 대응된다.

| Dockerfile | Pod 정의 파일 | 역할 |
|---|---|---|
| `ENTRYPOINT` | `command` | 실행될 프로그램 자체 |
| `CMD` | `args` | 프로그램에 넘길 기본 인자(커맨드라인에서 넘긴 값이 있으면 대체됨) |

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  containers:
    - name: ubuntu
      image: ubuntu-sleeper
      command: ["sleep2.0"]   # ENTRYPOINT를 덮어씀
      args: ["10"]            # CMD를 덮어씀 → 최종 실행: sleep2.0 10
```

- `command`를 지정하지 않으면 이미지의 `ENTRYPOINT`가, `args`를 지정하지 않으면 이미지의 `CMD`가 그대로 쓰인다.
- `command`가 `CMD`를 덮어쓰는 게 아니라 `ENTRYPOINT`를 덮어쓴다는 점이 헷갈리기 쉬운 포인트.

## 환경 변수

```yaml
spec:
  containers:
    - name: app
      image: myapp
      env:
        - name: APP_COLOR
          value: pink
```

- `env`는 배열이며, 각 항목은 `name`/`value` 쌍이다.
- 값을 직접 넣는 대신 `valueFrom`으로 ConfigMap/Secret을 참조할 수도 있다 (아래 참고).

## ConfigMap

민감하지 않은 설정값(key-value)을 Pod 정의 파일 밖으로 빼내 중앙에서 관리하는 오브젝트.

```bash
# 명령형(imperative)
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
kubectl create configmap app-config --from-file=app_config.properties

kubectl get configmaps
kubectl describe configmap app-config
```

```yaml
# 선언형(declarative) - config-map.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

Pod에 전체 주입:

```yaml
spec:
  containers:
    - name: app
      image: myapp
      envFrom:
        - configMapRef:
            name: app-config
```

- 단일 값만 주입하려면 `env[].valueFrom.configMapKeyRef`를, 파일로 마운트하려면 `volumes`로 ConfigMap을 볼륨화하면 된다.

## Secret

Secret은 ConfigMap과 사용법이 동일하지만, 민감 정보(비밀번호, 키 등)를 다루기 위한 오브젝트다.

```bash
kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_Password=mysql_password
kubectl create secret generic app-secret --from-file=app_secret.properties

kubectl get secrets
kubectl describe secret app-secret      # 값은 숨겨서 보여줌
kubectl get secret app-secret -o yaml   # base64로 인코딩된 값까지 확인 가능
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=            # echo -n 'mysql' | base64 로 인코딩한 값
  DB_Password: bXlzcWxfcGFzc3dvcmQ=
```

```bash
echo -n 'mysql' | base64          # 인코딩
echo -n 'bXlzcWw=' | base64 -d    # 디코딩
```

Pod 주입 방식은 ConfigMap과 동일 (`envFrom.secretRef` 또는 볼륨 마운트).

> **Secret은 "안전"이 아니라 "덜 위험"이다.** 값이 base64로 인코딩될 뿐 암호화되지 않으므로, base64 문자열을 아는 사람은 누구나 바로 디코딩할 수 있다. 실질적인 안전성은 운영 관행에서 나온다:
> - Secret 정의 파일을 소스 코드 저장소에 커밋하지 않는다.
> - etcd에 [저장 시 암호화(Encryption at Rest)](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)를 활성화한다.
> - Kubernetes 자체도 Secret을 필요로 하는 노드에만 전달하고, kubelet이 tmpfs(디스크가 아닌 메모리)에 저장하며, Pod가 삭제되면 로컬 사본도 함께 삭제하는 식으로 노출 범위를 줄인다.
> - 더 강력한 보안이 필요하면 HashiCorp Vault 같은 외부 시크릿 관리 도구를 고려한다.

## Multi-Container Pod

하나의 Pod 안에 컨테이너 여러 개를 함께 배치하는 패턴. 같은 생명주기(같이 생성/같이 종료), 같은 네트워크 네임스페이스(localhost로 서로 참조 가능), 같은 스토리지 볼륨을 공유하기 때문에 Pod 간 통신을 위해 별도로 Service나 볼륨 공유를 구성할 필요가 없다.

```yaml
spec:
  containers:
    - name: app
      image: myapp
    - name: log-agent   # 사이드카 컨테이너
      image: log-agent
```

대표적인 3가지 디자인 패턴:
- **Sidecar**: 로깅/모니터링 에이전트처럼 메인 컨테이너를 보조하는 컨테이너를 나란히 배치.
- **Adapter**: 메인 컨테이너의 출력을 표준화된 형식으로 변환.
- **Ambassador**: 메인 컨테이너 대신 외부 서비스와의 연결(프록시)을 담당.

(Adapter/Ambassador는 CKA보다는 CKAD 범위에 더 가까운 주제.)

## InitContainer

일반 컨테이너는 Pod 생명주기 동안 계속 살아있는 프로세스를 기대하지만, **딱 한 번 실행되고 완료되어야 하는 작업**(레포지토리 클론, 외부 서비스/DB가 뜰 때까지 대기 등)에는 InitContainer를 쓴다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command: ["sh", "-c", "until nslookup myservice; do echo waiting for myservice; sleep 2; done;"]
    - name: init-mydb
      image: busybox:1.28
      command: ["sh", "-c", "until nslookup mydb; do echo waiting for mydb; sleep 2; done;"]
```

- InitContainer는 메인 컨테이너가 시작되기 전에 **순차적으로(하나씩)** 실행되어 전부 완료돼야 한다.
- 여러 개를 정의하면 정의된 순서대로 하나씩 실행된다.
- InitContainer가 실패하면 Kubernetes는 성공할 때까지 Pod를 반복해서 재시작한다.

## Self-Healing

Kubernetes의 self-healing은 기본적으로 **ReplicaSet/ReplicationController**를 통해 구현된다 — Pod 안 애플리케이션이 죽으면 자동으로 재생성해서 항상 지정된 개수의 replica를 유지한다. 애플리케이션 자체의 헬스 체크(Liveness/Readiness Probe)는 CKAD 범위이므로 여기서는 다루지 않는다.

# 색인과 출처
- Certified Kubernetes Administrator with Practice Tests (KodeKloud/Udemy) — Configure Applications 섹션 (Commands, Commands and Arguments, Environment Variables, ConfigMaps, Secrets, Multi Container PODs, Multi-container PODs Design Patterns, InitContainers, Self Healing Applications)
- [Kubernetes 공식 문서 — Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Kubernetes 공식 문서 — Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
