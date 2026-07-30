- 도커에서 **용량 관리를 해주는데도 정체를 모르겠을 땐 그냥 전체를 날리고 재설치를 해주자**

```powershell
# 관리자 권한 실행
wsl --unregister Ubuntu-22.04
wsl --install -d Ubuntu-22.04
# 사용할 id 비번 입력
```

```sh
sudo apt update
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin awscli

sudo usermod -aG docker $USER # 권한 부여는 다시 켜야 적용됨
```

> 추가 : aws configure도 설정해놔야 함
```sh
aws configure
```