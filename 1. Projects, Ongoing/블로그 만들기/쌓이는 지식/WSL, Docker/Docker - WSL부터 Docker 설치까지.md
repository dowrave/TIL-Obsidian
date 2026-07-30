## 우분투, WSL 2 설치

1. `Windows Powershell`를 관리자 권한으로 실행, 아래의 명령어를 입력한다.
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

2. 재부팅한다.

3. `wsl --list --online`을 입력, 목록 중에서 사용하고자 하는 것을 설치한다
```powershell
wsl --install -d ubuntu-22.04
```
> 22.04 LTS 버전이 설치되며, 우분투까지 자동으로 실행되고 아이디와 비밀번호를 입력하는 란이 뜬다.

4. 파워쉘에서 `wsl -l -v`를 입력하면, `버전 1`로 설치되었음을 확인할 수 있다.  **버전 2로 바꾸려면**
```powershell
wsl.exe --update
wsl --set-version ubuntu-22.04 2 
```
> 아래 명령어만 바로 입력하면, `커널 구성 요소 업데이트`가 필요하다는 안내가 뜬다. 

## 도커 설치
- 이제부터는 우분투 내에서 작업을 진행한다.

```sh
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
> 1. 우분투에서 사용 가능한 패키지 목록 업데이트
> 2. 필요 패키지 설치 (HTTPS 이용)
> 3. Docker의 공식 GPG 키 추가, 패키지의 무결성 검증
> 4. Docker의 공식 저장소를 `apt` 소스 목록에 추가

- 패키지 목록 다시 업데이트 & 도커 Community Editon 설치
```sh
sudo apt update
sudo apt install docker-ce
```

- 도커 서비스 시작 및 자동 시작 설정
```sh
sudo systemctl start docker
sudo systemctl enable docker
```

- (선택) `sudo` 명령어 없이 사용자를 docker 그룹에 추가
```sh
sudo usermod -aG docker ${USER}
```
> 이후 시스템 로그아웃 및 로그인 필요

- Docker 설치 확인
```sh
docker run hello-world
```