- 대표적인 오픈소스 설치 & 서로 연동하여 사용하는 부분만을 주로 다룰 것
- 2022년 기준 아직 대표적인 표준 MLOps는 없음! 적절한 툴은 상황에 맞춰 취사선택할 것.

## 1. 쿠버네티스 실습 환경 구축
- 쿠버네티스 환경 구축 도구에는
	- 프로덕션 레벨 : `kubeadm`
	- 쉬운 버전 : `kubespray`, `kops`
	- 학습 목적 : `k3s`, `minikube`, `microk8s`, `kind`

- 여기서는 `kubeadm`, `k3s`, `minikube` 3가지 도구를 비교해볼 것
- 모든 기능을 사용하고 노드 구성도 하고 싶다면 `kubeadm`을 권장함

## 2. 사전에 설치할 것들
1. 포트포워팅을 위해 **클러스터**에 설치할 것들
```sh
sudo apt-get update
sudo apt-get install -y socat
```
- `socat`은 포트포워딩을 위한 패키지

2. 도커 설치
1) 도커에 필요한 APT 패키지 설치
```sh
sudo apt-get update && sudo apt-get install -y ca-certificates curl gnupg lsb-release
```
2) 도커의 공식 GPG key 추가
```ubuntu
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

3) 도커 설치
```sh
sudo apt install docker.io

# 실행 확인
docker version

# Active: failed가 뜬다면
sudo systemctl start docker
sudo systemctl enable docker
```

4) 도커 설치 확인
```sh
sudo docker run hello-world
```
- Hello from Docker! 뜨면 ㅇㅋ

5) Sudo 없이 docker 커맨드 사용하도록 권한 추가
```sh
sudo groupadd docker 
sudo usermod -aG docker $USER 
newgrp docker

# 결과 확인
docker run hello-world
```

6) `kubelet`의 정상 작동을 위해, 클러스터 노드에서 `swap`이라고 불리는 가상 메모리를 꺼둔다. (단일PC는 권장 X)
```sh
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 
sudo swapoff -a
```
- 클러스터 - 클라이언트가 **같은 데스크톱일 때 swap을 끄면 속도 저하**가 있을 수 있음

## 3. Kubectl 설치
- 클라이언트 노드에 설치해야 함

1) `kubectl v1.21.7` 다운
```sh
curl -LO https://dl.k8s.io/release/v1.21.7/bin/linux/amd64/kubectl
```

2) 파일 권한 & 위치 변경
```sh
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

3) 설치 확인
```sh
kubectl version --client

// 정상 출력 메시지
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.7", GitCommit:"1f86634ff08f37e54e8bfcd86bc90b61c98f84d4", GitTreeState:"clean", BuildDate:"2021-11-17T14:41:19Z", GoVersion:"go1.16.10", Compiler:"gc", Platform:"linux/amd64"}
```

## 4. minikube 설치
```sh
wget https://github.com/kubernetes/minikube/releases/download/v1.24.0/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 설치 확인 - 여기서 minikube는 위 경로 `/usr/local/bin/minikube`를 따름
minikube version
```
[[minikube start 실행 & 관련 에러들]]

### 클러스터 : 쿠버네티스 구축
```sh
sudo su # root유저로 접근(--driver 옵션 때문)

minikube start --driver=none --kubernetes-version=v1.21.7 --extra-config=apiserver.service-account-signing-key-file=/var/lib/minikube/certs/sa.key --extra-config=apiserver.service-account-issuer=kubernetes.default.svc
```

- 사용하지 않을 애드온 비활성화
```sh
minikube addons disable storage-provisioner 
minikube addons disable default-storageclass

# 확인
minikube addons list
```

- 에러가 너무 많이 떠서 우분투에 있는 도커랑 k8s를 지우고 쿠버네티스 실습을 진행했던 환경(배쉬 쉘)에서 세팅함

### 클라이언트 : 쿠버네티스 구축
- 마찬가지로 root user로 작업해야 함

- 클러스터 노드에서 `config` 확인
```sh
minikube kubectl -- config view --flatten
```

> 에러 : `unable to open tmp/juju-<...>: permission denied`
>> 해결 : 
>>`sudo rm -rf /tmp/juju-mk*`
>>`sudo rm -rf /tmp/minikube.*`

- (재설치 시 지나가도 됨)
- 클라이언트 노드에서 `.kube` 폴더 생성
```sh
mkdir -p /home/$USER/.kube
```

- 위 경로에 `config`(확장자명x) 파일을 만들고 내용물은 위에서 확인한 `config`을 넣음
```sh
vi /home/$USER/.kube/config
```

#### 쿠버네티스 모듈 설치(클라이언트)

#### Helm
- 쿠버네티스 패키지 관련 자원을 한 번에 배포하고 관리할 수 있게 돕는 `패키징 매니지 도구`
```sh
# 다운
wget https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz

# 압축 해제 
tar -zxvf helm-v3.7.1-linux-amd64.tar.gz

# 파일 위치 변경
sudo mv linux-amd64/helm /usr/local/bin/helm

# 설치 확인
helm help
```

#### Kustomize
- 역시 여러 쿠버네티스 리소스를 한 번에 배포하고 관리할 수 있게 도와주는 패키지 매니징 도구
```sh
# 다운(바이너리)
wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv3.10.0/kustomize_v3.10.0_linux_amd64.tar.gz

# 압축 해제 & 파일 위치 변경
tar -zxvf kustomize_v3.10.0_linux_amd64.tar.gz
sudo mv kustomize /usr/local/bin/kustomize

# 설치 확인
kustomize help
```

#### CSI Plugin : Local Path Provisioner
- `CSI Plugin`은 k8s의 스토리지를 담당하는 모듈
- `Local Path Provisioner`는 단일 노드 클러스터에서 쉽게 쓸 수 있음

- 설치 : k8s yaml 파일을 가져옴
```sh
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.20/deploy/local-path-storage.yaml

# 대충 namespace ~ configmap 까지 출력되면 ㅇㅋ
```

-  결과 `pod`이 Running인지 확인
```sh
kubectl -n local-path-storage get pod
```

- Default Storage Class로 변경
```sh
kubectl patch storageclass local-path  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# 정상 시 출력
# storageclass.storage.k8s.io/local-path patched

# 변경되었는지 확인
kubectl get sc
```

그래픽카드 드라이버는 설치 못함 : `No drivers Found` 가 떠서..
#### (선택) : Nvidia Driver 설치

1. 엔비디아 드라이버 설치
```sh
sudo add-apt-repository ppa:graphics-drivers/ppa sudo apt update && sudo apt install -y ubuntu-drivers-common 
sudo ubuntu-drivers autoinstall ## 여기서 막힘
sudo reboot
```
>막힌 부분 해결법 : [원문](https://www.nemotos.net/?p=5061)
> `sudo apt install alsa-base`
>> `ALSA`라는 프로그램을 설치하면 됨

> 근데 3번째 쉘을 실행했을 때 `No drivers Found for installation`이 뜸. 실행 환경에 그래픽카드가 없지 않는데도 말임(`MX250, GPU : GP108`)

2. 엔비디아 도커 설치
```sh
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \ sudo apt-key add - distribution=$(. /etc/os-release;echo $ID$VERSION_ID) 
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list 
sudo apt-get update 
sudo apt-get install -y nvidia-docker2 && sudo systemctl restart docker

# 실행 확인(GPU를 사용하는 도커 컨테이너)

sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

3. 엔비디아 도커를 deafult container runtime으로 설정

```sh
sudo vi /etc/docker/daemon.json
```
```json
{ "default-runtime": "nvidia", "runtimes": { "nvidia": { "path": "nvidia-container-runtime", "runtimeArgs": [] } } }
```

- 파일 변경 확인 후 도커 재시작
```sh
sudo systemctl daemon-reload 
sudo service docker restart
```

- 변경사항 반영 확인
```sh
sudo docker info | grep nvidia

# 대충 아래처럼 뜨면 설치된 거
mlops@ubuntu:~$ docker info | grep nvidia Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux nvidia runc Default Runtime: nvidia
```

4. Nvidia-Device-Plugin

1) nvidia-device-plugin daemonset 생성
```sh
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.10.0/nvidia-device-plugin.yml
```

2) 해당 pod이 RUNNING인지 확인
```sh
kubectl geet pod -n kube-system | grep nvidia
```

3) 노드 정보에 gpu가 사용 가능인지 확인
```sh
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
```