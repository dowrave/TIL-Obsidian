- 이 가이드에서 추천하는 시작법은 이거임
```sh
sudo su # root유저로 접근(--driver 옵션 때문)

minikube start --driver=none --kubernetes-version=v1.21.7 --extra-config=apiserver.service-account-signing-key-file=/var/lib/minikube/certs/sa.key --extra-config=apiserver.service-account-issuer=kubernetes.default.svc
```

- 한 번 실행이 이상하게 되었다면 이거부터 해주자
```sh
minikube delete
sudo rm -rf /tmp/juju-mk*
```

#### 충돌과 해결법

##### 1. `Exiting due to GUEST_MISSING_CONNTRACK`
```sh
apt install conntrack

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

##### 2. `System has not been booted with systemd as init system (PID 1)`
- 우분투 홈 `etc/wsl.conf`에 접근 (커맨드 `sudo -e etc/wsl.conf`로도 가능함), 아래 항목 추가
```conf
[boot]
systemd=true
```

##### 3. `Exiting due to HOST_JUJU_LOCK_PERMISSION: Failed to start host: boot lock: unable to open /tmp/juju-`
- 세팅 실패하고 `minikube start`하면 보게 되는 에러 같음
```sh
sudo rm -rf /tmp/juju-mk*
```
- 이거만 해줘도 되는 듯?


##### 4. `[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/`
1. `/etc/docker/daemon.json` 파일에 항목 추가(`root`권한 필요)
```json
{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"log-driver": "json-file",
	"log-opts": { "max-size": "100m"
	},
	"storage-driver": "overlay2"
}
```

2.
```sh
mkdir -p /etc/systemd/system/docker.service.d
```

3. 도커 재시작
```sh
systemctl daemon-reload
systemctl restart docker
kubeadm reset
kubeadm init # 아래 에러 속 에러 참고
```

### 에러 속 에러
```sh
kubeadm init

# 대충 이런 내용의 에러가 뜸
msg="getting status of runtime: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"

```

- 해결법
```sh
rm /etc/containerd/config.toml
systemctl restart containerd
kubeadm init
```
> 또 에러 : `error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.`
> ....


##### 5. Booting up control plane에서 렉
- 확실하지만 돌아오는 해결법 : [[linux docker 삭제 후 재설치]]
