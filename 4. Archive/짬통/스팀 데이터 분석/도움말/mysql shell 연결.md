### 문제
- `mysql shell`을 켠 상황
```sh
\sql 
CREATE DATABASE steam;
# ERROR : Not Connected
```

### 해결
```shell
\sql # js -> SQL 전환
\connect root@localhost # 루트 계정, 로컬호스트 접속
# 비밀번호 입력
```

