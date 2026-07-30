- 왜? : 더 싸니까

- 블로그인데 월 15$ 정도가 나가고 있음 - 내 서비스라고 하더라도 월 2만원이라고 하면 작은 돈은 아님. 이 중 대부분이 EC2 관련(1년 약정, VPC, EC2 - 기타) 요금.
- AI에게 이것저것 물어보니 LightSail이라는 게 있다고 함

### 요금제 선택
- 일단 기존 EC2의 `t2.micro`는 1GB 메모리, 1개의 vCPU를 제공함
- IPv4에서 이걸 만족하는 요금제는 7$이므로 이걸 선택

### 인스턴스 생성
- `Ubuntu 22.04 LTS`, OS만 설치
- 기존에 설치하던 방식이 이미 있으니까 그대로 사용함

### 정적 IP 할당
- `LightSail`의 좌측 메뉴 중 `Networking` 접근
- `Create static IP`를 눌러서 방금 만든 인스턴스에 붙이고 이름도 붙여준다
- 이 정적 IP는 무료임

### 방화벽 포트 열기
- 백엔드와 통신하기 위한 창구를 열어놓아야 함
- 인스턴스 - `Networking` - `IPv4 Firewall` 부분에서 `+Add Rule`로 아래 항목들을 추가
	- HTTPS : 443
	- Custom : 8000(Django 기본 포트, 수정했다면 바꿀 것)
	- MySQL : 3306(**외부에서 DB로 접근할 일이 있을 때 추가**)
		- **필요할 때에만 추가하고 지워주는 게 좋다.** 기본 설정이 `Any`라서 모두에게 노출되어 있기 때문임. 로컬 IP도 자주 바뀌고.

### 터미널 들어가서 설치
- 인스턴스의 우측 상단에 있는 터미널 아이콘으로 접근할 수 있음
	- `Putty`가 필요없다

```sh
# 1. 패키지 목록 업데이트
sudo apt-get update -y

# 2. 도커 및 필요한 도구들 한꺼번에 설치
sudo apt-get install docker.io docker-compose awscli -y

# 3. 도커 서비스 활성화
sudo systemctl start docker
sudo systemctl enable docker

# 4. 현재 사용자에게 도커 권한 부여
sudo usermod -aG docker ${USER}

# 5. 설정 적용을 위해 터미널 세션 갱신 (로그아웃 없이 바로 적용)
newgrp docker
```

### AWS 로그인
```sh
aws configure
```
이후 나타나는 정보들 입력. 내 경우는 별도로 저장해둔 파일이 있다.

> 이건 인스턴스를 만들 때 처음에 1번만 하면 됨


### 데이터 옮기기 : 백업 파일 만들기
- 기존 EC2에서는 볼륨에 데이터들을 관리하고 있었는데, LightSail로 옮겨와야 함. 
- 볼륨 자체를 복사할 순 없고 볼륨 내의 데이터를 추출해서 라이트세일의 볼륨으로 옮겨야 한다.

1. 백업 파일 만들기
- EC2 인스턴스에 접속해서 아래 명령어 입력
```sh
docker exec mysql-container mysqldump -u [userName] -p[password] [tableName] > [backupFileName].sql
```
> 저 **`-p` 뒤의 비밀번호는 붙여야 함**
> - `[password]`를 붙이는 부분까지 정상적으로 입력했는데 `1045` 에러가 발생했음 : 비밀번호를 전달받지 못했다는 이슈
> - (처음 보는 이슈인데) `mysqldump`는 `-p` 뒤에 비밀번호를 붙여서 써야 한다고 한다. ?? 더 웃긴 건 그게 진짜 됨.

2. EC2 컨테이너 -> 라이트세일로 쏘기
```sh
scp -i [라이트세일키.pem] backup.sql ubuntu@[라이트세일_정적IP]:~/
```
-  여기서 `라이트세일키.pem`은 라이트세일 인스턴스의 `SSH key`로 `pem` 파일을 얻은 다음, 그걸 열어서 복사
- `nano lightsail_key.pem`으로 컨테이너에서 pem 파일을 만들면 됨. 복붙하고 `ctrl + O -> 엔터 -> ctrl + X`
- `라이트세일_정적IP`는 라이트세일을 만들 때 나왔던 `Public IPv4`
- 이게 정상적으로 동작했다면 아래 로그가 나온다. `yes` 입력.
```
The authenticity of host '' can't be established.
ED25519 key fingerprint is SHA256:~~~
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

- 라이트세일에서 확인
- 그냥 메인 화면에서 `ls` 쳐보면 됨
```
ubuntu@ip-172-26-0-60:~$ ls
blogBackup_260127.sql // 잘 들어왔음
```

> 참고) 저 블로그 DB 파일은 대충 2mb 정도였다.
> 스팀 DB를 옮겨놨음. 얘는 1GB 정도 된다.

### MySQL 설치
- (우선 개인적으로 있던 docker compose 파일 내부에 있는 백업 파일을 넣는 동작은 동작하지 않게 수정)
- 스왑 메모리 설정
	- 스팀 DB의 1GB 정도 되는 파일을 MySQL에 밀어넣기 위한 준비
```sh
// 라이트세일
sudo swapon --show // 스왑 확인. 나오는 게 없으면 스왑 없음

// SSD에 2GB 크기의 빈 파일을 만듦
sudo fallocate -l 2G /swapfile

// 권한 설정 및 스왑 포맷(스왑 공간 선언)
sudo chmod 600 /swapfile
sudo mkswap /swapfile

// 스왑 활성화
sudo swapon /swapfile

// 확인 
sudo swapon --show

// NAME      TYPE SIZE USED PRIO
// /swapfile file   2G   0B   -2

// 재부팅 시에도 유지 설정 (=/etc/fstab 파일 끝에 설정을 추가함)
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

// 추가 설정 : SSD 수명 및 성능을 고려해 Swappiness 조절
// 스왑 메모리를 얼마나 적극적으로 사용할지를 결정하는 값

// 현재 값 확인
cat /proc/sys/vm/swappiness

// 10으로 변경
sudo sysctl vm.swappiness=10

// 영구 반영을 위해 설정 파일 수정
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

이외 로직은 로컬에서 이미지 빌드하고 ECR로 푸시하고 다시 Pull해서 실행시키면 됨
그런데 백업 파일을 넣어야 함
```cs
# 1. 데이터베이스 생성
docker exec -it mysql-container mysql -u root -p'root_password' -e "CREATE DATABASE IF NOT EXISTS blog_db; CREATE DATABASE IF NOT EXISTS steam_db;"

# 2. 데이터 복원
cat ~/blogBackup_260127.sql | docker exec -i mysql-container mysql -u root -p'root_password' blog
cat ~/steam_data_backup.sql | docker exec -i mysql-container mysql -u root -p'root_password' steam
```
> DB를 만드는 과정은 필수임

### Django 설치
- `settings.py`에서 lightsail의 정적 IP주소 추가 
- + 밑에서 사용할 LightSail을 위한 징검다리 도메인 `origin-api.{}.com` 추가
```python
ALLOWED_HOSTS = [
	# Lightsail의 정적IPv4주소 추가,
	"origin-api.{}.com",
	# ...
]
```

나머지 이미지 작업은 MySQL과 동일함

### 클라우드프론트 -> 라이트세일 연결

#### Route 53
- **`origin-api` 서브 도메인을 이용한 레코드**를 하나 더 추가해줌
	- **클라우드프론트에서 라이트세일의 IPv4 주소를 직접 Origin Domain 필드에 입력할 수 없기 때문**에, 이 역할을 대신 해주는 서브 도메인을 Route 53을 이용해 구성해주는 방식임
	- 기존엔 백엔드는 `api`라는 서브 도메인을 썼음

#### 클라우드 프론트
1. **원본 탭 : `origin-api`라는 서브 도메인을 추가**
2. **동작 탭 : 기존의 `api` 로 보내지는 요소들을 원본 `origin-api`로 보내도록 수정**

### 최종 실행
![[Pasted image 20260127185249.png]]
> 사이트도 잘 나타나고 LightSail 콘솔에서 도커 로그 띄워보면 잘 실행되는 걸 볼 수 있음

리뷰 페이지의 이미지, 첨부된 동영상, 음악 등등도 잘 동작함

### 이제 EC2는 꺼둘 거임





