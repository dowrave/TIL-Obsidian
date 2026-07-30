![[ingress.webp]]- 한 클러스터에서 여러 서비스를 운영할 때 외부 연결에 대한 것이다. `NodePort`는 서비스 개수만큼 포트를 오픈하고 사용자에게 어떤 포트인지 일일이 알려줘야 한다.
- **여러 서비스를 소수의 포트로 사용**하는게 `Ingress`이다. 
### Ingress 만들기
- `echo` 웹 앱을 버전별로 도메인을 다르게 지정하여 만듦

- `minikube ip`로 테스트 클러스트의 IP를 구하고 도메인 주소로 사용함
	- ex) `192.168.64.5`라면 도메인은 `v1.echo.192.168.64.5.sslip.io`
- 참고) `sslip.io`는 IP주소를 도메인에 넣어 바로 사용할 수 있게 해줌(도메인 테스트에도 여러 설정이 필요함)

### minikube에 Ingress 활성화하기
- Pod, ReplicaSet, Deployment, Service와 달리 별도의 컨트롤러가 필요하다. 여기선 `nginx ingress controller`를 사용함
	- 이외에도 `haproxy`, `traefik`, `alb` 등이 있다. 
```sh
minikube addons enable ingress

# ingress controller 확인
kubectl -n ingress-nginx get pod
```

- 실행 결과
```
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-qt4pl        0/1     Completed   0          14m
ingress-nginx-admission-patch-2z2wp         0/1     Completed   1          14m
ingress-nginx-controller-5959f988fd-4lp4z   1/1     Running     0          14m
```

- 설정 확인
	- 도커 드라이버 사용 X 
	```
	curl -I http://{minikube ip로 얻은 ip}/healthz
	```
	- 도커 드라이버 사용 O
	```
	minikube service ingress-nginx-controller -n ingress-nginx --url
	```
	- 위 명령어로 창을 띄운 상태가 되고 그 상태에서는 명령어를 입력할 수 없다(배쉬 쉘이 아님)

따라서 **아래 명령어는 별도의 Shell을 띄운 다음에 입력**한다 
```shell
curl -I http://{1번째 주소}/healthz
```
- 결과
```
TP/1.1 200 OK
Date: Wed, 21 Dec 2022 05:27:45 GMT
Content-Type: text/html
Content-Length: 0
Connection: keep-alive
```

> 참고) 위에서 띄운 Ingress는 `kubectl get all`에 포함되지 않는다!


### echo 웹 앱 배포
- 2가지 버전을 배포하기로 했음
- `Ingress Spec`의 `rules.host` 부분을 `minikube ip`로 변경함
>Docker Driver는 `rules.host` 부분을 `127.0.0.1`을 사용한다. 
- `echo-v1.yml`, `echo-v2.yml`
`echo-v1.yml`(v2는 v1인 값을 모두 v2로 바꿔주면 ㅇㅋ - `apiVersion`의 `v1`은 유지)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-v1
spec:
  rules:
    - host: v1.echo.127.0.0.1.sslip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: echo-v1
                port:
                  number: 3000

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: echo
      tier: app
      version: v1
  template:
    metadata:
      labels:
        app: echo
        tier: app
        version: v1
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
          livenessProbe:
            httpGet:
              path: /
              port: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: echo-v1
spec:
  ports:
    - port: 3000
      protocol: TCP
  selector:
    app: echo
    tier: app
    version: v1
```

- 실행
```sh
kubectl apply -f echo-v1.yml,echo-v2.yml

# 상태 확인
kubectl get ingress
kubectl get ing
```

- 접속 확인
```sh
curl -I v1.echo.127.0.0.1.sslip.io:PORT
curl -I v2.echo.127.0.0.1.sslip.io:PORT
```
{호스트 정보}:{포트 정보}를 입력함
- {호스트 정보}는 `Docker Driver`을 사용한다면 `127.0.0.1` 고정인 듯
- `PORT`는 `ingress-nginx-controller` 서비스의 1번째 항목이다.


## Ingress 생성 흐름
> 1. `Ingress Controller`는 `Ingress 변화`를 체크
> 2. `Ingress Controller`는 변경된 내용을 `Nginx`에 설정하고 프로세스 재시작

- YAML로 만든 Ingress 설정을 nginx 설정으로 바꾸는 것일 뿐이며, 이를 자동으로 Ingress Controller가 하는 것이다. 
- 도메인, 경로 연동 이외에도 요청 timeout, 요청 max size 등 다양한 프록시 설정을 할 수 있다.

### 실습
- 아래 조건을 만족하는 Ingress를 만드시오
| 키                | 값           |
| ----------------- | ------------ |
| Deployment 이름   | nginx        |
| Deployment LAbel  | app: nginx   |
| Deployment 복제수 | 3            |
| Container 이름    | nginx        |
| Container 이미지  | nginx:latest |
| Ingress 도메인    | nginx.xxx.sslip.io             |

- `my_nginx.ingress.yml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
    - host: nginx.127.0.0.1.sslip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 3000 # 80으로
---
# Deployment
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
          image: nginx
          livenessProbe:
            httpGet:
              path: /
              port: 3000 # 80으로 변경


---
# Service
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
    - port: 3000  # 80으로
      protocol: TCP
  selector:
    app: nginx

```
- 연결 확인
	- 터미널 1) `minikube service ingress-nginx-controller -n ingress-nginx --url`
	- 터미널 2) 터미널 1로 나온 1번째 포트를 이용해 `curl -I nginx.127.0.0.1.sslip.io:{포트}` 접속 시도해보자

- `502 bad gateway`가 뜬다든가, `CrashLoopBackOff`가 뜬다든가 하는 이슈가 있었음
	- 포트 이슈인 듯 함 : `port: 3000 -> port: 80`으로 바꿔주니까 해결됨
