### Docker Compose
- 지금까지는 도커를 커맨드라인에서 명령어로 작업했다. 간단한 작업이므로 명령어가 길지 않지만 설정할 게 많아질수록 명령어가 복잡해질 것이다.
- **도커는 복잡한 설정을 쉽게 관리하기 위해 `YAML`방식의 설정 파일을 이용한 `Docker Compose`라는 툴을 제공한다.**

#### 설치
- `WSL`을 설치했다면 자동으로 됨 ㅅㄱ

#### wordpress 만들기
- 빈 디렉토리에 `docker-compose.yml` 파일을 만들어 설정을 입력함 (파일명은 강제인 듯)
```
$ mkdir wp
$ cd wp
$ vi docker-compose.yml

# docker-compose.yml 내용 #########
version: '2'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:
  wp_data:
###########################
```
- `vi` 에디터에서 참고할 거
	- `vi`로 들어가면 정상 모드
	- `i`를 누르면 삽입모드 <-> `esc`로 정상모드
	- 파일 저장 : `:w` (종료는 하지 않음)
	- 파일 저장 & 종료 : `:wq` (항상 버퍼를 파일에 기록 & 수정시간 업데이트) / `:x` (변경사항이 있는 경우에만 버퍼를 파일에 씀)
	- 파일 안저장 & 종료 : `!wq`

`docker-compose.yml` 경로에서 이걸 실행하면 됨
```
docker-compose up
```
- 이거 켜면 입력창이 돌아오지 않는 게 정상인 것 같음

- Docker-Compose에 대한 공부는 숙제로 남겨둠
