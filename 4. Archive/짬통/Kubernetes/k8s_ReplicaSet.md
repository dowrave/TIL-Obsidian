- Pod만을 단독으로 만들면 Pod에 문제가 생겼을 때 자동으로 복구되지 않는다.
- **`ReplicaSet`은 Pod을 정해진 수만큼 복제하고 관리한다.**

## ReplicaSet 만들기
`echo-rs.yml`
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: echo-rs
spec:
  replicas: 1 # 유지할 Pod의 개수
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```
- 생성 :  `kubectl apply -f echo-rs.yml`
- 리소스 확인 : `kubectl get po,rs` (pod, ReplicaSet)
```
NAME                READY   STATUS    RESTARTS   AGE
pod/echo-rs-xqxh7   1/1     Running   0          33s

NAME                      DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-rs   1         1         1       33s
```

## ReplicaSet 설명
![[rs.webp]]
- `Label을 체크` -> `원하는 수`의 Pod이 없으면 새로운 Pod을 생성함
| 정의            | 설명            |
| --------------- | --------------- |
| `spec.selector` | label 체크 조건 |
| `spec.replicas` | 원하는 Pod 개수 |
| `spec.template` | 생성할 Pod의 명세                |
```yaml
  template: # spec.template
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```
- 즉 `spec.template` 내부 내용은 이전의 `pod` 설정 파일과 완전히 동일하다!

- `Pod`의 label 확인 : `kubectl get pod --show-labels`
```
NAME            READY   STATUS    RESTARTS   AGE    LABELS
echo-rs-xqxh7   1/1     Running   0          4m4s   app=echo,tier=app
```
- `app=echo,tier=app`

- 여기서 `label`을 임의로 제거하면
```
# app- 지정하면 app label 제거
kubectl label pod/echo-rs-xqxh7 app-

kubectl get pod --show-labels
```
- **기존 pod**에서 **LABELS의 app이 제거**되었다.
- 한편 **또다른 pod이 실행**되었다. 왜냐하면 1개의 app을 유지해야 하는데 그게 없어져 버렸기 때문이다. 이게 `ReplicaSet`이 하는 일이다.

#### ReplicaSet의 작동 원리
> 1. ReplicaSet Controller는 조건을 감시하면서 현재 태와 원하는 상태가 다른지를 체크
> 2. 원하는 상태가 되도록 Pod을 생성하거나 제거함
> 3. Scheduler는 API 서버를 감시하면서 할당되지 않은 Pod을 체크함
> 4. Scheduler는 할당되지 않은 Pod을 감지하고, 적절한 노드에 배치한다.
> 5. 이후 노드는 기존대로 동작한다.

- 즉 `ReplicaSet`은 `ReplicaSet Controller`가 관리하고, `Pod`의 할당은 `Scheduler`가 관리한다. 

## 스케일아웃
- Pod을 쉽게 여러 개로 복제할 수 있다.
- `echo-rs-scaled.yml`
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: echo-rs
spec:
  replicas: 4 # <- 요 옵션만 늘려줌
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```

```
NAME                READY   STATUS    RESTARTS   AGE
pod/echo-rs-764cs   1/1     Running   0          6m8s
pod/echo-rs-b8bzm   1/1     Running   0          13s
pod/echo-rs-d724d   1/1     Running   0          13s
pod/echo-rs-mmzzf   1/1     Running   0          13s
pod/echo-rs-xqxh7   1/1     Running   0          11m

NAME                      DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-rs   4         4         4       11m

```
- label을 체크한다고 했음 : 가장 실행이 오래된 건 위에서 `app=echo`를 제거한 Pod임
- 위 실행을 제거하려면 `replicaset` 부분만 지워주면 된다 : `kubectl delete replicaset.apps/echo-rs`

#### 마무리
- ReplicaSet은 원하는 개수의 Pod을 유지하는 역할이다.
- Label을 이용하여 Pod을 체크하므로, Label이 겹치지 않게 신경써서 정의해야 한다.
- ReplicaSet을 단독으로 쓰는 경우는 없으며, Deployment에서 ReplicaSet을 이용한다.

#### 실습
- 다음 ReplicaSet을 만드시오
| 키                  | 값           |
| ------------------- | ------------ |
| Replicaset 이름     | nginx        |
| Replicaset Selector | app: nginx   |
| ReplicaSet 복제수   | 3            |
| Container 이름      | nginx        |
| Container 이미지    | nginx:latest |
- 내 답변 : `my_nginx.yml`
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels: # 얘가 조건항
      app: nginx # tier는 없어도 됨
  template: # 여기는 들어가는 컨테이너들 
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```