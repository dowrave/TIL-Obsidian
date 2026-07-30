- 쿠버네티스에서 가장 널리 쓰이는 오브젝트
- [[k8s_ReplicaSet]]을 이용해 [[k8s_Pod]]을 업데이트하고 이력을 관리해 롤백하거나 특정 버전으로 돌아갈 수 있음

- `echo-deployment.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
spec:
  replicas: 4
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
- `kind`, `metadata` 부분을 빼면 ReplicaSet과 완전히 동일함
```sh
kubectl apply -f echo-deployment.yml

# 리소스 확인
kubectl get po,rs,deploy
```
- `yml`파일에 경로 잘못 설정해서 오류 발생 : [[k8s_ImagePullBackOff]]

- 결과
```
# 결과
NAME                               READY   STATUS    RESTARTS   AGE
pod/echo-deploy-68b9dfd874-5wl42   1/1     Running   0          10s
pod/echo-deploy-68b9dfd874-kjgjs   1/1     Running   0          9s
pod/echo-deploy-68b9dfd874-rvvwg   1/1     Running   0          16s
pod/echo-deploy-68b9dfd874-t2gc6   1/1     Running   0          16s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23h

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-deploy   4/4     4            4           16m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-deploy-68b9dfd874   4         4         4       16s
replicaset.apps/echo-deploy-6c8bb996     0         0         0       16m 
# 잘못 실행된 Replicaset인 듯 : 안 지워지네?

```

- 모두 Running이 될 때까지 기다린 다음 `echo-deployment-v2.yml`을 만든다(`spec.template.spec.containers.image`의 값만 `ghcr.io.subicura/echo:v2`로 바꿔줌)
- `kubectl apply -f echo-deployment-v2.yml`
- `kubectl get po,rs,deploy`

- 결과
```
# Pod Name이 전부 바뀌었음
NAME                               READY   STATUS    RESTARTS   AGE
pod/echo-deploy-58cfb87569-2ngz8   1/1     Running   0          29s
pod/echo-deploy-58cfb87569-7z2b5   1/1     Running   0          28s
pod/echo-deploy-58cfb87569-jfxtn   1/1     Running   0          23s
pod/echo-deploy-58cfb87569-nm2cc   1/1     Running   0          24s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23h

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-deploy   4/4     4            4           20m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/echo-deploy-58cfb87569   4         4         4       29s
replicaset.apps/echo-deploy-68b9dfd874   0         0         0       3m44s
replicaset.apps/echo-deploy-6c8bb996     0         0         0       20m
```
--> Pod이 모두 새로운 버전으로 바뀌게 됨  
(정확한 의미는 **새로운 Pod을 생성하고, 기존 Pod을 제거**함)
- 특기할만 한 건 업데이트 전의 `replicaset`이 남아있다는 거 정도?


#### 업데이트 동작 원리
- `Deployment`는 새로운 이미지로 업데이트하기 위해 `ReplicaSet`을 이용한다. 
- 업데이트가 완료되면 새로운 `ReplicaSet`을 생성하고, 해당 `ReplicaSet`이 새로운 버전의 `Pod`을 생성한다.

![[deploy-1.webp]]
- 이렇게 2개의 다른 버전의 `ReplicaSet`이 놓이게 되면, 새로운 `ReplicaSet`에서 1개의 `Pod`을 생성한다
	- 이게 정상적으로 작동하면 기존 ReplicaSet의 Pod을 1개 줄인다.(예제에선 4->3)
![[deploy-2.webp]]

- 위 과정을 기존 버전의 Pod은 0개, 새로운 버전의 Pod은 Replica의 갯수가 될 때까지 반복한다.

### 각 컨트롤러의 동작 방식
> 1. `Deployment Controller`는 Deployment 조건을 감시 & 현재 상태와 원하는 상태가 다른 것을 체크
> 2. `Deployment Controller`가 원하는 상태가 되도록 `ReplicaSet` 설정
> 3. `ReplicaSet Controller`는 `ReplicaSet` 조건을 감시 & 현재 상태와 원하는 상태가 다른 것을 체크
> 4. `ReplicaSet Controller`가 원하는 상태가 되도록 `Pod`을 생성하거나 제거
> 5. `Scheduler`는 API 서버를 감시, 할당되지 않은 `Pod` 체크
> 6. `Scheduler`는 할당되지 않은 새로운 `Pod`을 감지 & 적절한 `Node`에 배치
> 7. 이후 `Node`는 기존대로 동작
- `Deployment`는 `Deployment Controller`가 관리, `ReplicaSet`과 `Pod`은 기존`Controller`, `Scheduler`가 관리한다.


### 버전 관리 `kubectl rollout`
- `Deployment`는 변경된 상태를 기록한다.
```sh
# 히스토리 확인
kubectl rollout history deploy/echo-deploy

# revision 1 히스토리 상세 확인
kubectl rollout deploy/echo-depeloy --revision=1

# 바로 전으로 롤백
kubectl rollout undo deploy/echo-deploy

# 특정 버전으로 롤백
kubectl rollout undo deploy/echo-deploy --to-revision=2
```

- 왜 조회가 안될까?

### 배포 전략 설정
`echo-strategy.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy-st
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 3
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
          livenessProbe:
            httpGet:
              path: /
              port: 3000
```
- 이전과 차이점
	- `RollingUpdate` 방식을 사용할 때 동시에 업데이트하는 `Pod`의 개수를 (기본)25% -> 3개로 변경함



- 생성 & 결과 확인
```sh
kubectl apply -f echo-strategy.yml
kubectl get po,rs,deploy

# 이미지 변경 (명령어로)
kubectl set image deploy/echo-deploy-st echo=ghcr.io/subicura/echo:v2

# 이벤트 확인
kubectl describe deploy/echo-deploy-st
```

- 결과
```sh
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m19s  deployment-controller  Scaled up replica set echo-deploy-st-5694b4995 to 4
  Normal  ScalingReplicaSet  10s    deployment-controller  Scaled up replica set echo-deploy-st-5c7d4b8c4 to 3
  Normal  ScalingReplicaSet  10s    deployment-controller  Scaled down replica set echo-deploy-st-5694b4995 to 1 from 4
  Normal  ScalingReplicaSet  9s     deployment-controller  Scaled up replica set echo-deploy-st-5c7d4b8c4 to 4 from 3
```

- 참고)  `maxSurge`와 `maxUnavailable`의 기본값은 25%로, 대부분의 상황에서 적당하다. 

## 실습
1. Deployment 만들기
| 키               | 값         |
| ---------------- | ---------- |
| 이름             | nginx      |
| Label            | app: nginx |
| Replica          | 3          |
| Container 이름   | nginx      |
| Container 이미지 | nginx:1.14.2           |

`depl_nginx.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
```
- 컨테이너의 `label은` `spec.template.metadata.labels`에 입력한다.

2. `replicas`를 5로 조정한다.
- `depl_nginx_v2.yml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 5 # <- 요거만 만들어줌
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
```

```
kubectl apply -f depl_nginx_v2.yml
```

3. 이미지를 `nginx:1.19.5`로 변경한다.
- 다른 이미지의 정보를 돌아가고 있는 이미지에 전달하는 방식이 본문에서 한 거란 말임? `kubectl set image deploy/echo-deploy-st echo=ghcr.io/subicura/echo:v2`

- 그러면 echo에 전달하는 값만 바꿔주면 되는 거 아님? / 그래도 파일로 관리하는 게 좋으려나?
`kubectl set image deploy/nginx nginx=nginx:1.19.5`
`
- `yml` 파일로 전달하는 방식이 좋아 보인다 : 명령어로 전달하는 건 `ReplicaSet` 수준의 관리를 어떻게 할 지 모르겠네 -  `replicas` 같은 거

- 된다 : 확인은  `nginx describe deploy/nginx` 이거로 
```
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.19.5
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
```
- 바뀐 거 볼 수 있음
- 근데 왜 `kubectl rollout history`는 동작 이상하게 하냐
