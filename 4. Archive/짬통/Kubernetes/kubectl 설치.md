- `kubectl`은 쿠버네티스 CLI 도구로, **쿠버네티스 클러스터에 명령어를 전달하는 방법**이다.

### 설치
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.5/bin/windows/amd64/kubectl.exe
```
- 윈도우 쉘이 아니라 `Git Bash`에 명령어를 치고 있음을 잊지 맙시다

#### 설치 확인
- 참고 : **`Git Bash`는 관리자 권한으로 실행**되어야 함
```sh
kubectl version --short
```
- `kubectl version`도 무방하나 deprecate될 예정

