- `스왑 메모리`라는게 있다고 함
	- 램이 부족할 때 스토리지를 메모리처럼 사용하는 공간.
	- EBS는 어차피 남아도니까 크게 상관도 없다.
	- **느리다.** 메모리가 넘칠 것에 대비한 임시용이지 계속해서 사용할 목적으로 설정하는 게 아니다.

```sh
# 우분투에서 작업 진행

# 1GB 스왑 파일 생성 (원하는 용량으로 변경 가능)
sudo fallocate -l 1G /swapfile

# 파일 권한 변경
sudo chmod 600 /swapfile

# 스왑 영역 생성
sudo mkswap /swapfile

# 스왑 활성화
sudo swapon /swapfile

# 적용 확인
swapon -s
free -m

```