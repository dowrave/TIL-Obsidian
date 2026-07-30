- kubectl의 역할 : 쿠버네티스의 상태를 확인하고, 원하는 상태를 요청한다. 컨테이너 로그도 확인하고 원격으로 접속할 수 있다.
- 배울 건 `kubectl`의 명령어 + 오브젝트(`pod`, `ReplicaSet`, `Deployment`, `Service`)의 사용법

## kubectl 명령어
| 명령어     | 설명                                                        |
| ---------- | ----------------------------------------------------------- |
| `apply`    | 원하는 상태를 적용한다. 보통 `-f` 옵션으로 파일과 함께 사용 |
| `get`      | 리소스 목록을 보여준다.                                     |
| `describe` | 리소스의 상태를 자세하게 보여준다.                          |
| `delete`   | 리소스를 제거한다                                           |
| `logs`     | 컨테이너의 로그를 본다                                      |
| `exec`     | 컨테이너에 명령어를 전달한다. 컨테이너에 접근할 때 사용함.  |
| `config`   | kubectl 설정을 관리한다.                                    |


> `alias`로 명령하기
> `kubectl`이 오타가 자주 나기 때문에 `k`로 줄여쓰면 편함

```sh
# alias 설정
alias k='kubectl'

# shell 설정 추가
echo "alias k='kubectl'" >> ~/.bashrc
source ~/.bashrc
```

#### 상태 설정하기`apply`
```sh
kubectl apply -f [파일명, URL]
```
- 원하는 리소스의 상태를 `YAML`로 작성하고 `apply` 명령어로 선언한다.
- 예시 : URL로 워드프레스 배포
```sh
kubectl apply -f https://subicura.com/k8s/code/guide/index/wordpress-k8s.yml
```


#### 리소스 목록 보기`get`
```sh
kubectl get [TYPE]
```
- 옵션이 다양하게 있음
	- `-o` : 출력 형태 변경
	- `--show-labels` : 레이블 확인
```sh
# Pod만 조회
kubectl get pod

# 줄임말, 복수형 사용 가능 - 모두 pod
kubectl get pods
kubectl get po

# 여러 Type도 가능
kubectl get pod,service
kubectl get po,svc

# Pod, ReplicaSet, Deployment, Service, Job 조회
kubectl get all

# 결과 포맷 변경
kubectl get pod -o wide # 기본
kubectl get pod -o yaml
kubectl get pod -o json

# Label을 추가로 조회
kubectl get pod --show-labels
```

#### 리소스 상태 보기`describe`
```sh
kubectl describe [TYPE]/[NAME]

kubectl describe [TYPE] [NAME]
```
- 특정 리소스의 상태가 궁금하거나 생성이 실패한 ㅣㅇ유를 확인할 떄 주로 사용함
- ex)
```sh
kubectl get pod

# 위에서 얻은 type(pod)/id를 아래에 넣음
kubectl describe pod/wordpress-mysql-7d7ccf6fdc-fzkfm 
```

#### 리소스 제거`delete`
```sh
kubectl delete [TYPE]/[NAME]
kubectl delete [TYPE] [NAME]
```
- 쿠버네티스에서 선언된 리소스 제거, 사용 문법은 describe와 동일
> 참고) Pod은 제거해도 계속 살아난다. ReplicaSet이 Pod의 갯수를 유지하기 때문이다. 
- 여러 개 지우고 싶으면 띄어쓰기하고 다른 거 입력하면 됨

#### 컨테이너 로그 조회`logs`
```sh
kubectl logs [POD_NAME]
```
- 옵션
	- `-f` : 실시간 로그
	- `-c` : 한 pod에 여러 컨테이너가 있을 경우
- ex)
```sh
kubectl get pod
kubectl logs wordpress-sdoifhsdf
```

#### 컨테이너 명령어 전달`exec`
```sh
kubectl exec [-it] [POD_NAME] -- [COMMAND]
```
- `-it` : 쉘로 접속 & 컨테이너 상태 확인
- `-c` : 여러 컨테이너가 있다면 이 옵션으로 컨테이너를 지정함

- ex) 워드프레스 컨테이너 접속
```sh
kubectl get pod
kubectl exec -it {container_id} -- bash 
```

#### 설정 관리`config`
- `kubectl`은 여러 쿠버네티스 클러스터를 `컨텍스트Context`로 설정하고 필요에 따라 선택할 수 있다.
- 현재 설정된 컨텍스트 확인 & 원하는 컨텍스트 지정
```sh
# 현재 컨텍스트 확인
kubectl config current-context

# 컨텍스트 설정
kubectl config use-context minikube
```

#### 그 외
```sh
# 전체 오브젝트 종류 확인
kubectl api-resources

# 특정 오브젝트 설명
kubectl explain {object}
```

#### 마무리
- 리소스 제거
```sh
kubectl delete -f https://subicura.com/k8s/code/guide/index/wordpress-k8s.yml
```
