- 컨테이너에서 설정 파일을 관리하는 방법은 이미지 빌드 시 복사, 컨테이너 실행 시 외부 파일 연결 등이 있다.
- k8s는 `ConfigMap`으로 설정을 관리한다.

### ConfigMap 만들기
- `config-file.yml`
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    metrics_path: /prometheus/metrics
    static_configs:
      - targets:
          - localhost:9090
```

- 위 파일을 설정으로 만든다.
```sh
# ConfigMap 생성
kubectl create cm my-config --from-file=config-file.yml

# ConfigMap 조회
kubectl get cm

# ConfigMap 내용 상세 조회
kubectl describe cm/my-config
```

- 생성한 `ConfigMap`을 `/etc/config` 디렉토리에 연결
- `alpine.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["sleep"]
      args: ["100000"]
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: my-config
```

- 배포 및 확인
```sh
kubectl apply -f alpine.yml

# 접속 & 설정 확인
kubectl exec -it alpine -- ls /etc/config
kubectl exec -it alpine -- cat /etc/config/config-file.yml
```
> 저 `/etc/config`의 위치가 왜 `git bash` 위치로 뜨지;
>> 이전에 했던 `hostpath` 켜고 하니까 잘 작동함
>> 근데 `hostpath.yml` 꺼도 잘 작동함
>> `/etc/config`를 `etc/config`로 바꿔도 잘 작동함
> 안되다가 갑자기 되니까 당황스러운데; 모르겠다

- env 파일로 만들기
`config-env.yml`
```yaml
hello=world
haha=hoho
```

- env 포맷으로 만듦
```sh
kubectl create cm env-config --from-env-file=config-env.yml

kubectl describe cm/env-config
```
- 결과
```
Name:         env-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
haha:
----
hoho
hello:
----
world

BinaryData
====

Events:  <none>
```

### YAML 선언하기
- ConfigMap을 YAML 파일로 정의하기
- `config-map.yml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  hello: world
  kuber: netes
  multiline: |-
    first
    second
    third
```

- 적용 후 마운트 내용 확인하기
```sh
kubectl delete cm/my-config

# configmap 생성
kubectl apply -f config-map.yml

# alpine 적용
kubectl apply -f alpine.yml

# 적용 내용 확인
kubectl exec -it alpine -- cat /etc/config/multiline

# 만약 경로를 못찾으면
kubectl exec -it alpine -- sh
$ cat /etc/config/multiline
```

### ConfigMap을 환경변수로 사용
-  Volume이 아님!
- `alpine-env.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-env
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["sleep"]
      args: ["100000"]
      env:
        - name: hello
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: hello
```

- 환경변수 확인하기
```sh
kubectl apply -f alpine-env.yml

kubectl exec -it alpine-env -- env
```