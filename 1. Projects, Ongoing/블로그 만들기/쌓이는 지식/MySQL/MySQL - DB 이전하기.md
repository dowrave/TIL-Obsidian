- 아래 쉘을 돌릴 때, `Windows PowerShell`은 비추천한다 : `backup.sql` 파일을 만드는 과정에서 `utf-8` 설정을 하더라도 글씨가 깨져서 나타난다. 내 경우 `git bash`을 사용했다.

---

### 1단계: 로컬 MySQL 데이터베이스 백업

로컬 데이터베이스에서 데이터의 백업 파일을 생성합니다. 이 작업은 `mysqldump` 유틸리티를 사용하여 수행할 수 있습니다.

```sh
mysqldump -u [username] -p [database_name] > backup.sql
```
- `[username]`은 로컬 MySQL 데이터베이스의 사용자 이름입니다.
- `[database_name]`은 백업하려는 데이터베이스의 이름입니다.
- `backup.sql`은 생성될 백업 파일의 이름입니다. 이 파일에는 데이터베이스 구조와 데이터가 포함됩니다.

> 1. `mysqldump`은 윈도우 기준 `mysql`을 설치할 때 `program files`의 `MySQL server x.x` 폴더의 `bin` 폴더에 있다. 
> 	- 인식하지 못한다면 Path에 환경변수로 해당 경로를 추가하고 쉘을 껐다 키거나, 아예 컴퓨터를 재부팅한다.
> 2. 위에서도 언급했듯, `Windows PowerShell`로 실행했을 때 `utf-8` 인코딩을 설정하더라도 해당 파일이 깨져서 뒤의 작업이 제대로 이뤄지지 않을 수 있다.

> 컨테이너에 접근할 경우, 추가로 `-P`(포트) 와, `-h`(호스트)를 추가해서 전달한다.
```sh
mysqldump -u [username] -p -h [host] -P [port] [database_name] > [backupfile_name].sql
```


### 2단계: 백업 파일을 컨테이너로 복사

생성된 백업 파일을 Docker 컨테이너 내부로 복사합니다. 이 작업은 `docker cp` 명령어를 사용하여 수행할 수 있습니다.

```sh
docker cp backup.sql [container_id]:/backup.sql
```

- `[container_id]`는 목표 MySQL 컨테이너의 ID 또는 이름입니다.

### 3단계 : backup.sql을 컨테이너 내부에서 실행하기
- 크게 2가지 방법이 있다.

1. 컨테이너를 생성한 다음 들어가서 실행하는 방법
```sh
docker exec -it [container_id] bash
mysql -u [username] -p [database_name] < /backup.sql
```

2. 아예 `docker-compose.yml` 파일에 명시하는 방법
```sh
services: 
  db:
    image: mysql:8.0.36
    volumes:
      - ./backup.sql:/docker-entrypoint-initdb.d/backup.sql
```

> 로컬의 `backup.sql` 파일을 컨테이너 내부의 `docker-entrypoint-initdb.d/backup.sql`로 마운트한다는 의미로, 해당 폴더는 **MySQL 컨테이너 이미지에 자체적으로 내장**되어 있다고 한다. 
> - 해당 폴더에 넣으면 최초로 컨테이너를 만들 때 해당 폴더 내부의 스크립트들이 자동으로 실행된다. 이외에 별도로 명시할 것이 없다.
> - 주의할 점으로, 기존에 마운트한 다른 볼륨이 있다면 이를 삭제하고 `docker compose up`을 진행해 완전 초기화 상태에서 컨테이너가 생성되도록 해야 한다.

