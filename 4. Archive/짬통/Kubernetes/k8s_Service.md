- `Pod`은 자체 IP를 갖고 다른 Pod과 통신할 수 있으나 쉽게 사라지고 생성되므로 직접 통신하는 것을 권장하지 않는다.
- 대신, **k8s는 별도의 고정된 IP를 가진  `Service`를 만들고 그 서비스를 통해 Pod에 접근하는 방식을 사용**한다.
![[pod-service.webp]]

## Service(ClusterIP) 만들기
- `ClusterIP`는 클러스터 내부에 새 IP를 만들고, 여러 개의 Pod을 바라보는 LoadBalancer 기능을 제공한다. 
- 서비스 이름을 내부 도메인 서버에 등록, Pod 간에 서비스 이름으로 통신할 수 있다.

- `counter-redis-svc.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: counter
      tier: db
  template:
    metadata:
      labels:
        app: counter
        tier: db
    spec:
      containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
              protocol: TCP

---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  ports:
    - port: 6379
      protocol: TCP
  selector:
    app: counter
    tier: db
```

> `---`
> 	한 개의 yaml파일에 여러 개의 리소스를 정의할 땐 `---`를 **구분자**로 사용한다.

```
NAME                         READY   STATUS    RESTARTS   AGE
pod/redis-5cb78b9855-jwg9m   1/1     Running   0          43s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    24h
service/redis        ClusterIP   10.99.209.251   <none>        6379/TCP   43s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1/1     1            1           44s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-5cb78b9855   1         1         1       44s

```
- `redis deployment`와 `redis service`가 생성되었음을 볼 수 있다. 
- 같은 클러스터에서 생성된 Pod은 `redis`라는 도메인으로 `redis pod`에 접근할 수 있다. (`redis.default.svc.cluster.local`로도 접근할 수 있다. 서로 다른 `namespace`와 `cluster`를 구분할 수 있다. )

- `ClusterIP Service`의 설정

| 정의                    | 설명                                          |
| ----------------------- | --------------------------------------------- |
| `spec.ports.port`       | 서비스가 생성할 Port                          |
| `spec.ports.targetPort` | 서비스가 접근할 Pod의 Port(기본: Port와 동일) |
| `spec.selector`         | 서비스가 접근할 Pod의 Label 조건                                              |

- `redis Service`에서 정의된 `spec.selector`은 `deployment`에서 정의된 `spec.selector`의 label 정보이므로, 해당 Pod을 가리키게 된다. 이 Pod의 6379 포트로 연결한다.

- `redis`에 접근할 `counter` 앱을 `deployment`로 만든다.
- `counter-app.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  selector:
    matchLabels:
      app: counter
      tier: app
  template:
    metadata:
      labels:
        app: counter
        tier: app
    spec:
      containers:
        - name: counter
          image: ghcr.io/subicura/counter:latest
          env:
            - name: REDIS_HOST
              value: "redis"
            - name: REDIS_PORT
              value: "6379"
```

- `counter app pod` -> `redis app pod` 접근 확인하기
```sh
kubectl apply -f counter-app.yml

# counter app에 접근하기
kubectl get pod
kubectl exec -it COUNTER NAME -- sh

# counter 컨테이너 내부
curl localhost:3000
curl localhost:3000
telnet redis 6379

# redis app 내부
dbsize
KEYS *
GET count
quit

# counter 내부
exit
```

### Service 생성 흐름
> 1. `Endpoint Controller`는 `Service`, `Pod`을 감시, 조건에 맞는 `Pod`의 IP 수집
> 2. `EndPoint Controller`가 수집한 IP로 `Endpoint` 생성
> 3. `Kube-Proxy`는 `EndPoint` 변화를 감시, 노드의 `iptables`을 설정
> 4. `CoreDNS`는 `Service`를 감시, 서비스 이름과 IP를 `CoreDNS`에 추가함

- `iptables`는 커널 레벨의 네트워크 도구
	- 여러 IP에 트래픽을 전달함
	- 규칙이 많아지면 느려지는 이슈가 있어 `ipvs`를 쓰는 옵션도 있다.
- `CoreDNS`는 클러스터 내부용 도메인 네임 서버
	- IP 대신 도메인 이름을 사용하게 해줌 
	- 클러스터에서 호환성을 위해 `kube-dns`라는 이름으로 생성됨

- `Endpoint` : 서비스의 접속 정보를 가지고 있음
	- 상태 확인하기
```sh
kubectl get endpoints
kubectl get ep

# 출력
NAME         ENDPOINTS           AGE
kubernetes   192.168.49.2:8443   24h
redis        172.17.0.3:6379     21m
```
```
# redis Endpoint 확인
kubectl describe ep/redis

# 출력
(...)
Subsets:
  Addresses:  172.17.0.3 <---- Redis Pod의 IP
(...)
```
- `Endpoint Addresses` 정보에는 `Redis Pod`의 IP가 보이며, `Replicas`가 여러 개라면 여러 IP가 보인다.

## Service(NodePort) 만들기
- **`ClusterIP`는 클러스터 내부에서만 접근**할 수 있다. 
- **클러스터 외부에서 접근할 수 있도록 NodePort를 만든다.**
- `counter-nodeport.yml`
```
apiVersion: v1
kind: Service
metadata:
  name: counter-np
spec:
  type: NodePort
  ports:
    - port: 3000
      protocol: TCP
      nodePort: 31000 # 노드에 오픈할 포트
  selector:
    app: counter
    tier: app
```
- `spec.ports.nodePort` : 미지정시 30000~32768 중 자동할당. 노드에 오픈되는 포트이다.
```sh
kubectl apply -f counter-nodeport.yml

# 서비스 상태 확인
kubectl get svc
```

- 테스트 클러스터의 노드 IP를 구하고 31000으로 접근
```sh
minikube ip # 테스트 클러스터의 노드 ip
curl 192.168.64.4:31000

# Docker Driver 사용자(와따시)
minikube service counter-np
```
```
# 출력

| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | counter-np |        3000 | http://192.168.49.2:31000 |
|-----------|------------|-------------|---------------------------|
* counter-np 서비스의 터널을 시작하는 중
| NAMESPACE |    NAME    | TARGET PORT |          URL           |
|-----------|------------|-------------|------------------------|
| default   | counter-np |             | http://127.0.0.1:52345 |
|-----------|------------|-------------|------------------------|
```

![[nodeport-multi.webp]]
- `NodePort`는 클러스터의 모든 노드에 포트를 오픈한다.
- 여러 노드가 있다면 아무 노드로 접근해도 지정한 Pod으로 접근할 수 있다.
- 참고 ) `NodePort`는 `ClusterIP`의 기능을 **기본으로 포함**한다.

## Service(LoadBalancer) 만들기
- NodePort의 단점 : 노드가 사라졌을 때 자동으로 다른 노드를 통한 접근이 불가능하다는 점이다.
- 예를 들면 3개의 노드가 있을 때, 아무 노드로 접근해도 `NodePort`로 연결할 수는 있으나 어떤 노드가 살아있는지는 알 수 없다.
- **자동으로 살아 있는 노드에 접근하기 위해 모든 노드를 바라보는 `LoadBalancer`가 필요**하다. 
	- 브라우저는 노드 포트가 아니라 LoadBalancer에 요청하고, `LoadBalancer`가 알아서 살아 있는 노드에 접근하면 노드 포트의 단점을 없앨 수 있다.
- `counter-lb.yml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: counter-lb
spec:
  type: LoadBalancer
  ports:
    - port: 30000
      targetPort: 3000
      protocol: TCP
  selector:
    app: counter
    tier: app
```
```sh
kubectl apply -f counter-lb.yml

kubectl get svc

# 결과
coutner-lb   LoadBalancer   10.97.11.242    <pending>     30000:30096/TCP   6s
```
- `EXTERNAL-IP` : `<pending>`임을 확인할 수 있다. 
- `LoadBalancer`는 AWS, Google Cloud, Azure 등의 클라우드 환경이 아니라면 사용이 제한적이다.
- 특정 서버(노드)를 가리키는 무언가(LB)가 필요한데, 이 무언가가 가상머신이나 로컬 서버에는 존재하지 않는다.

### minikube에 가상 LB 만들기
- `MetalLB` :  LB를 쓸 수 없는 환경에서 가상 환경을 만들어준다. 
	- 현재 떠 있는 노드를 LB로 설정한다.
```sh
minikube addons enable metallb

# ip 확인 -> ConfigMap으로 지정해야 함
minikube ip

minikube addons configure metallb
# 입력하는 두 ip 모두 위에서 확인한 ip 입력하면 됨
```

- 혹은 `minikube`없이 직접 `ConfigMap`을 작성할 수도 있다.
- `metallb-cm.yml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.64.4/32 # minikube ip
```
```sh
kubectl apply -f metallb-cm.yml

kubectl get svc

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)           AGE
counter-np   NodePort       10.100.44.155   <none>         3000:31000/TCP    14m
coutner-lb   LoadBalancer   10.97.11.242    192.168.49.2   30000:30096/TCP   6m
kubernetes   ClusterIP      10.96.0.1       <none>         443/TCP           25h
redis        ClusterIP      10.99.209.251   <none>         6379/TCP          41m

```
- `counter-lb`의 `192.168.49.2:30000`으로 접근해본다.
	- `docker driver`는 `minikube service counter-lb`로 접속
- `LoadBalancer`도 `NodePort`의 기능을 기본적으로 포함한다

### 마무리
- **서비스는 `Low-Level` 수준의 네트워크를 이해하고, 성능 & 보안 이슈를 신경써야 한다. 파고들면 어려운 영역이라서 일단 대충 설명되었음**
- 실제로는 `NodePort`와 `LoadBalancer`를 제한적으로 사용한다.
	- 보통 웹앱을 배포하면 `80` 또는 `443` 포트를 사용, 하나의 포트에서 여러 서비스를 도메인이나 경로에 따라 다르게 연결하기 때문이다.
	- 이 부분은 `Ingress`에서 다룬다.

## 문제
1. `echo` 서비스를 `NodePort`로 `32000`포트로 오픈하기
| 키                 | 값                       |
| ------------------ | ------------------------ |
| Deployment 이름  | echo                     |
| Deployment Label | app: echo                |
| Deployment 복제수  | 3                        |
| Container 이름     | echo                     |
| Container 이미지   | ghcr.io/subicura/echo:v1 |
| NodePort 이름      | echo                     |
| Nodeport Port      | 3000                     |
| NodePort NodePort  | 32000                    |
- `echo_svc.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echo
  template:
    metadata:
      labels: 
        app: echo
    spec:
    containers:
    - name: echo
      image: ghcr.io/subicura/echo:v1
      ports:
        - containerPort: 3000
          protocol: TCP
---
apiVersion: v1
kind: NodePort # <-- NodePort가 들어갔다면 Service를 이걸로 바꿔줘야 함
metadata:
  name: echo
spec:
  ports:
    - port: 3000
      protocol: TCP
      nodePort: 32000
  selector: # <- Deployment와 맞춰줘야 함(matchLabels까진 필요 없나봄)
	app: echo
```
- 틀린 것들
1. `Service` 부분
	1) Label을 찾게 해주려면 `spec.selector` 부분을 지정해줘야 함
	2) `Service "echo" is invalid: spec.ports[0].nodePort: Forbidden: may not be used when type is 'ClusterIP'` - `type: Service` 값을 `type: NodePort`로 바꿔주면 해결
2. `Deployment`부분 
	1) Label을 지정하는 부분은 `spec.selector.matchLabels` 부분과 `spec.template.metadata.labels`  으로 2가지이다.
	2) `metadata`에는 `name`으로 들어가지만 `apiVersion` 값에 `apps`이 있다면 `label의 key`값은 `app`으로 들어가는 것 같다. 
			- `Service`에서도 `app: echo`로 레이블 지정한다