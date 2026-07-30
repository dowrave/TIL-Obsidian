- 쿠버네티스를 OS에 설치하기 위해
	- 최소 3대의 마스터 서버
	- 컨테이너 배포를 위한 n개의 노드 서버
가 필요하다. 
- 처음 공부할 때는 과정이 복잡하고 배포 환경(AWS, Google Cloud, Azure, Bare Metal 등)에 따라 방법이 다르기 때문에 어려움

- 여기서는 개발 환경을 위해 **마스터 + 노드를 하나의 서버에 설치**하여 쉽게 관리하는 방법을 이용함

- 개발 환경 구축 방법 : `minikube`, `k3s`, `docker for desktop`, `kind` 등

> 참고 : 개발환경과 운영환경의 차이점
> 개발환경은 단일 노드로 여러 노드에 스케줄링하는 테스트가 어렵고, `LoadBalancer`와 `Persistent Local Storage` 또한 가상으로 만들어야 한다. 
> 따라서 실습을 정확하게 하려면 운영환경(멀티노드)에서 진행해야 한다.

### minikube
- 쿠버네티스 스케줄러에는 `scheduler`, `controller`, `api-server`, `etcd`, `kubelet`, `kube-proxy` 등을 설치해야 하며, 필요에 따라 `dns`, `ingres controller`, `storage class`등을 설치해야 한다. 
- `minikube`는 이런 설치를 쉽고 빠르게 해줌

##### 설치
[minikube-installer.exe](https://github.com/kubernetes/minikube/releases/latest/download/minikube-installer.exe) 다운
- Hyper-V는 Windows 10 - Home 버전엔 없어서 패스, virtualBox에서 진행함

- 설치 확인
```sh
minikube version

# 가상머신 설정(관리자 권한 실행 필요)
minikube start --driver=hyperv 
# hyperV 오류 발생시 virtualbox 이용
minikube start --driver=virtualbox

# docker for windows가 깔려 있다면 그냥 위 2개는 무시하고 아래로 가도 됨(깔려 있는 경우 docker가 기본 가상머신이 되기 때문)

# k8s 실행
minikube start --kubernetes-version=v1.23.1

# 상태 확인
minikube status

# ssh 접속
minikube ssh # docker@minikube:~$로 들어가짐

# ip 확인
minikube ip

# 정지
minikube stop

# 삭제
minikube delete
```

### Docker Desktop을 이용할 경우 서비스 노출
```sh
minikube service wordpress
```
- `wordpress` 설치를 따로 해줘야 함 : 일단 스킵

-------------------

### k3s
- 별도 클라우드 서버에 `k3s`를 설치해서 원격으로 실습할 수도 있으나 `minikube`가 잘 설치되기 때문에 스킵

##### docker for desktop
- 여기 자체에서 쿠버네티스 클러스터를 활성화할 수도 있다.
-  설정 -> `Kubernetes` -> `Enable Kubernetes`
- 리소스를 많이 잡아먹기 때문에 실습 때는 `minikube`를 사용하자.

-------------------

#### minikube 고-급 기능

1. 다중노드 구성
```sh
# 단일 노드
minikube start

# 다중 노드
minikube start -n 2 
```

2. 멀티 프로필
- 1개의 개발 PC에 여러 minikube를 쓸 수 있음
```sh
# 가상머신 1
minikube start # 기본 profile - minikube로 생성

# 가상머신 2
minikube start -p helloworld # helloworld라는 이름의 profile로 생성

# 프로필 목록
minikube profile list

# 현재 사용중인 프로필
minikube profile

# 다른 profile로 변경 (Active)
minikube profile helloworld
minikube profile minikube

# 가상머신 제거
minikube delete
```

### 그 외 유용한 도구들
- [kubectx](https://github.com/ahmetb/kubectx) : 컨텍스트 전환 CLI
- [kubens](https://github.com/ahmetb/kubectx) : 네임스페이스 전환 CLI
- [k9s](https://github.com/derailed/k9s) : 클러스터 관리 CLI
- [kubespy](https://github.com/pulumi/kubespy) : 쿠버네티스 상태 실시간 확인
- [Lens](https://k8slens.dev/) : 클러스터 관리 CLI
