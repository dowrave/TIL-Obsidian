- **쿠버네티스에서 관리하는 가장 작은 배포 단위**.
- 도커와의 차이점
	- **도커는 컨테이너**를 만든다
	- k8s는 Pod을 만든다. **Pod이란 개념은 1개 이상의 컨테이너를 포함**한다.


#### Pod 빠르게 만들기
```sh
kubectl run echo --image ghcr.io/subicura/echo:v1
# pod/echo created
```
- 참고 : `k8s v1.18` 이상은 `run`이 `Pod`을 만들지만 이 이하는 `Deployment`를 만든다.

```
kubectl get pod
```
- `STATUS`는 컨테이너가 정상적으로 생성되면 Running으로 바뀌고, 오류가 있다면 에러를 표시한다.

- Pod 상태 더 상세하게 확인
```sh
kubectl desribe pod/echo
```
- `describe` 명령어는 해당 리소스의 상세한 정보를 알려준다. 
	- 주로 `Events`를 확인하게 되는데, 현재 Pod의 상태를 이벤트별로 확인할 수 있다.
```
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  108s  default-scheduler  Successfully assigned default/echo to minikube
  Normal  Pulling    106s  kubelet            Pulling image "ghcr.io/subicura/echo"
  Normal  Pulled     85s   kubelet            Successfully pulled image "ghcr.io/subicura/echo" in 20.4301379s
  Normal  Created    85s   kubelet            Created container echo
  Normal  Started    84s   kubelet            Started container echo

```

### Pod 생성 분석
- 클러스터 `minikube` 내부에 `Pod`이 있고, Pod 내부에 `컨테이너`가 있다.
![[pod-single.webp]]

> Pod의 생성 과정
>> 1. `Scheduler`는 API 서버 감시, 할당되지 않은 `Pod`을 체크함
>> 2. `Scheduler`는 할당되지 않은 `Pod`을 적절한 노드에 할당함(`minikube`는 단일노드)
>> 3. `kubelet` : 노드에 설치되어 있으며, 자신에게 할당된 `Pod`을 체크
>> 4. `kubelet`은 `Scheduler`에 의해 자신에게 할당된 `Pod`의 정보를 확인 & 컨테이너 생성
>> 5. `kubelet`은 자신에게 할당된 `Pod`의 정보를 `API 서버에 전달함`
> 이렇듯 노드가 많아지더라도 `Scheduler`만 잘 작동하면 문제가 없는 구조다.

```
kubectl delete pod/echo
```

#### YAML로 설정파일(Spec) 작성하기
- `kubectl run`은 실무에선 거의 안 쓴다.  
- 설정이 복잡하고 다양한데, 명령어로만 표현하면 금방 복잡해지고 관리하기 어렵기 때문.
- 이를 위해 `yaml`파일로 복잡한 내용을 표현 & 변경 내용을 버전으로 관리할 수 있다.
- `echo-pod.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo
  labels:
  app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
```
- `run` 명령어와의 차이점은, `label`이 추가되었다는 것이다.
- `k8s`는 리소스 관리 시 `name`, `label`을 이용한다.

- **리소스 정의 4가지 필수 요소**
| 정의       | 설명          | 예시                                      |
| ---------- | ------------- | ----------------------------------------- |
| `version`  | 오브젝트 버전 | v1, app/v1, networking.k8s.io/v1 등..     |
| `kind`     | 종류          | Pod, ReplicaSet, Deployment, Service, ... |
| `metadata` | 메타데이터    | name, label, annotation(주석) 등으로 구성 |
| `spec`     | 상세명세      | 리소스 종류마다 다르다 |
>`version`
>>쿠버네티스 버전에 따라 지원하는 리소스의 버전이 다르기 때문에 실습 시 유의해야 함
>>>`Alpha(v1alpha1, ....)`
>>>`Beta(v1beta1, ...)`
>>>`Stable(v1)`

```sh
kubectl apply -f echo-pod.yml

# 생성까지 시간 좀 걸림
kubectl get pod

kubectl logs echo
kubectl logs -f echo # 로그 실시간 조회

# Pod 컨테이너 접속
kubectl exec -it echo -- sh

# 컨테이너 내부에서
ls # 파일 리스트
ps # 현재 돌아가고 있는 프로세스
exit

# Pod 제거
kubectl delete -f echo-pod.yml
```

### 컨테이너 상태 모니터링
- `컨테이너 생성`과 `서비스 준비`는 약간의 차이가 있다.
- 서버를 실행하면 짧게 수초, 길게 수분의 초기화 시간이 필요하며, **실제 접속이 가능할 때 서비스가 준비되었다**고 할 수 있다.
- 쿠버네티스는 초기화하는 동안 서비스되는 것을 막을 수 있다.

#### livenessProbe
- **컨테이너가 정상적으로 동작되는지 체크, 아니라면 컨테이너를 재시작하여 문제를 해결**한다.
- 체크 방식은 여러 가지가 있으나, 여기서는 `http get` 요청 확인으로 체크함

- `echo-lp.yml`
```yml
apiVersion: v1
kind: Pod
metadata:
  name: echo-lp
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      livenessProbe:
        httpGet:
          path: /not/exist
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 2 # Default 1
        periodSeconds: 5 # Defaults 10
        failureThreshold: 1 # Defaults 3
```
- 의도적으로 존재하지 않는 `path(/not/exist)`와 `port(8080)`을 입력함
- 파일 생성 후 `kubectl apply -f echo-lp.yml` 입력
- 이후 `kubectl get pod` 입력
```
$ kubectl get pod
NAME      READY   STATUS             RESTARTS      AGE
echo-lp   0/1     CrashLoopBackOff   2 (12s ago)   33s
```
- 정상적인 응답이 없었기 때문에 Pod이 여러 번 시되었고, `CrashLoopBackOff` 상태로 변경되었다.
>상태체크는 `httpGet` 외에도 `tcpSocket`, `exec` 방법 등으로 체크할 수 있음

#### readinessProbe
- 컨테이너 준비되었는지 체크, 정상적인 준비가 되지 않았다면 Pod으로 들어오는 요청을 제외함
- `livenessProbe`와의 차이점은, 문제가 있어도 Pod을 재시작하지 않고 요청만 제외한다는 것이다.
- `echo-rp.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo-rp
  labels:
   app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      readinessProbe:
       httpGet:
         path: /not/exist
         port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 2
      periodSeconds: 5
      failureThreshold: 1
```
- `kubectl get pod` 입력 시 `STATUS - Running`이 뜸 / 그러나 `READY`는 `0/1`

#### livenessProbe + readinessProbe
- 보통은 이 둘을 같이 적용함
- `echo-pod-health.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: echo-health
 labels:
   app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      livenessProbe:
        httpGet:
          path: /
          port: 3000
      readinessProbe:
        httpGet:
          path: /
          port: 3000
```
- `3000번` 포트와 `/` 경로는 정상적이므로 Pod은 오류 없이 생성됨 
- `STATUS - Runing`, `READY - 1/1` 확인할 것
	- 여기서 `1/1`의 숫자 의미는 컨테이너 개수인 것 같다
#### 다중 컨테이너
- 1개의 Pod은 여러 컨테이너를 가질 수 있다고 했다.
- `counter-pod-redis.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
  labels:
    app: counter
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/counter:latest
      env:
        - name: REDIS_HOST
          value: "localhost"
    - name: db
      image: redis
```
- 요청 횟수를 `redis`에 저장하는 간단한 웹앱을 다중 컨테이너로 생성함
> 환경변수 설정
>> 환경변수 `env` 정의는 `name`과 `value`를 별도로 정의한다.

- 같은 Pod에 컨테이너가 생성되므로 `counter` 앱은 redis를 `localhost`로 접근할 수 있다. 
```sh
# pod 생성
kubectl apply -f counter-pod-redis.yml

kubectl get pod

# 로그 확인
kubectl logs counter 
# 단독 사용시 오류 발생 : Pod 속의 컨테이너까지 지정해줘야 한다. 

kubectl logs counter app
kubectl logs counter db


# Pod의 app 컨테이너 접속
kubectl exec -it counter -c app -- sh
$ curl localhost:3000
$ curl localhost:3000
$ telnet localhost 6379 # 접속인 듯?
$$ dbsize
$$ KEYS *
$$ GET count
quit

# Pod 제거
kubectl delete -f counter-pod-redis.yml
```

## 실습
- 다음 조건을 만족하는 Pod을 만드시오
1. 
| 키               | 값         |
| ---------------- | ---------- |
| Pod 이름         | mongodb    |
| Pod Label        | app: mongo |
| Container 이름   | mongodb    |
| Container 이미지 | mongo:4           |
- 특이한 점 : `yml`파일엔 대문자 못 씀
- `mongodb.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
  labels:
    app: mongo
spec:
  containers:
    - name: mongodb
      image: mongo:4
```
2.
| 키                 | 값           |
| ------------------ | ------------ |
| Pod 이름           | mariadb      |
| Pod Label          | app: mariadb |
| Container 이름     | mariadb      |
| Container 이미지   | mariadb:10.7 |
| Container 환경변수 | MYSQL_ROOT_PASSWORD:123456             |
- 트러블 슈팅 : `Error from server (BadRequest): error when creating "mariadb.yml": Pod in version "v1" cannot be handled as a Pod: strict decoding error: unknown field "metadata.label"` 오류가 뜬 이유 : `labels`를 `label`이라고 적어서
`mariadb.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mariadb
  labels:
    app: mariadb
spec:
  containers:
    - name: mariadb
      image: mariadb:10.7
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
```
- 특이한 건 `value` 값이 숫자여도 `""`가 들어간다는 거

- 끝나고 `kubectl delete pod/mariadb pod/mongodb` 해주자. 