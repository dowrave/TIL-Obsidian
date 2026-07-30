 도커는 **컨테이너의 상태를 그대로 이미지로 저장한다.**
	-  어떤 앱을 이미지로 만든다면, 리눅스만 설치된 컨테이너 <- 애플리케이션 설치 <- 이 상태를 그대로 이미지로 저장함
	- 콘솔에서 명령어를 입력하는 것과 별 차이가 없지만, 좋은 샘플이 많다 (깃허브에서 `dockerfile` 검색)
	- 또한, 컨테이너의 가벼운 특성 + 레이어 개념으로 생성 & 테스트를 빠르게 수행할 수 있다.


### Sinatra  웹 앱 샘플
- `Sinatra`는 가벼운 웹 프레임워크임

- 아래 두 파일을 생성함
- `Gemfile`
```
source 'https://rubygems.org'
gem 'sinatra'
gem 'thin'
gem 'puma'
gem 'reel'
gem 'http'
gem 'webrick'
```
- `app.rb`
```rb
require 'sinatra'
require 'socket'

get '/' do
  Socket.gethostname
end
```
- `Gemfile`은 패키지 관리, `app.rb`는 호스트명을 출력하는 웹 서버를 만듦

- 패키지 설치 & 서버 실행 (`루비 설치 X시 실행 X`)
```
bundle install 
bundle exec ruby app.rb
```

- ruby 설치 : docker를 이용함 (도커 내에서 루비 실행)
```
docker run --rm \
-p 4567:4567 \
-v $PWD:/usr/src/app \
-w /usr/src/app \
ruby \
bash -c "bundle install && bundle exec ruby app.rb -o 0.0.0.0"
```
- 옵션
	- `-v` : 데이터 볼륨 : 컨테이너의 파일은 호스트에 바로 저장됨
	- `-w` : 컨테이너 내의 프로세스가 실행될 디렉토리 설정


- 호스트의 디렉토리를 루비가 설치된 컨테이너의 디렉토리에 마운트 & 그대로 명령어를 실행함
- 이렇게 하면 로컬에 개발 환경을 구축하지 않고 **도커 컨테이너를 개발 환경으로 사용할 수 있음**
	- 위의 명령어는 `ruby` 컨테이너 내에서 수행되는 거니까..
- `5e280ab1e172` 같은 도커 컨테이너의 호스트명이 보이면 성공임(`docker ps`에 나오는 컨테이너 id와 같은 값임)

### Dockerfile 만들기
- 도커는 이미지를 만들기 위해 `Dockerfile`이라는 이미지 빌드용 DSL(`Domain Specific Language`) 파일을 사용함. 단순 텍스트 파일로, 소스와 함께 관리됨.
- 일단 리눅스 서버에서 테스트로 설치해보고 될 때까지 최적의 과정을 `Dockerfile`로 작업해야 함. 
- 일단 `Ruby` 웹앱을 `ubuntu`에 배포하는 과정부터 살펴보면
| 순서 | 작업 |
| ---- | ---- |
| 1    | 우분투 설치     |
| 2    |   ruby 설치    |
| 3    |     소스 복사 |
| 4    |    Gem 패키지 설치  |
| 5     |   Sinatra 서버 실행
이고, 아래 스크립트로 실행시킨다.
```bash
# 1. ubuntu 설치
apt-get update

# 2. ruby 설치
apt-get install ruby
gem install bundler

# 3. 소스 복사
mkdir -p /usr/src/app
scp Gemfile app.rb root@ubuntu:/usr/src/app 

# 4. Gem 패키지 설치
bundle install

# 5. Sinatra 서버 실행
bundle exec ruby app.rb
```

- 위의 과정이 정상적으로 실행되었다면, 이 과정을 Dockerfile로 만들면 된다.
- 파일명 : `dockerfile`
```dockerfile
# 1. ubuntu 설치
FROM ubuntu:16.04
MAINTAINER subicura@subicura.com
RUN apt-get -y -update

# 2. 우분투 설치
RUN apt-get -y install ruby
RUN gem install bundler

# 3. 소스 복사
COPY . /usr/src/app

# 4. Gem 패키지 설치 (실행 디렉토리 설정)
WORKDIR /usr/src/app
RUN bundle install

# 5. Sinatra 서버 실행 (Listen 포트 정의)
EXPOSE 4567 
CMD bundle exec ruby app.rb -o 0.0.0.0
```
- 핵심 명령어는 파일을 복사하는 `COPY`와 명령어를 실행하는 `RUN`이다. 
- 위의 쉘 스크립트와 비교해도 내용이 거의 동일하다.
	- 차이점이라면 도커 빌드 중엔 키보드 입력이 불가하므로, `(y/n)` 질문 방지를 위한 `-y` 옵션을 추가한 것 정도이다.

### Docker Build
- `Dockerfile`을 통해 이미지를 만드는 과정이다.

```
docker build [OPTIONS] PATH | URL | -
```
- 생성할 이미지 이름을 지정하기 위한 `-t(--tag)`  옵션만 알면 빌드가 가능하다.

- `Dockerfile`을 만든 디렉토리로 이동, 아래 명령어를 입력한다.
```
docker build -t app .
```
- `Successfully built ~~~~` 메시지가 보이면 정상적으로 이미지를 생성한 것이다.
```
docker image
```

### dockerfile 기본 명령어

#### FROM
- 베이스 이미지를 지정한다. 
```dockerfile
FROM <image>:<tag>
# EX)
FROM ubuntu:16.04
```
- 어떤 이미지든 베이스 이미지가 될 수 있으며, 반드시 베이스 이미지가 필요하다.
- `tag`는 될 수 있으면 `latest`라는 기본값보다 구체적인 버전을 지정하는 것이 좋다.
- 다양한 베이스 이미지는 `Docker Hub`에서 확인할 수 있음.

### MAINTAINER
```dockerfile
MAINTAINER <name>

# 예시
MAINTAINER subicura@subicura.com
```
- Dockerfile을 관리하는 사람의 이름 or 이메일 정보를 적는다. 빌드에 영향을 주지 않음.

### COPY
```dockerfile
COPY <src>... <dest>

# EX
COPY . /usr/src/app
```
- 파일이나 디렉토리를 이미지로 복사한다.
- 일반적으로 소스 복사에 이용하며, `target` 디렉토리가 없다면 자동으로 생성한다.

#### ADD
```dockerfile
ADD <src>... <dest>

# ex
ADD . /usr/src/app
```
- `copy`와 유사하나 `src`에 파일 대신 URL을 입력할 수 있고, 압축 파일을 입력하는 경우 자동으로 압축 해제 후 복사된다.

#### RUN
```dockerfile
RUN <command>
RUN ["executable", "param1", "param2"]
RUN bundle install
```
- 명령어를 그대로 실행한다. 
- 내부적으로 `/bin/bash -c` 뒤에 명령어를 실행하는 방식
	- `/bin/sh -c` 라고 나와있는데 짤린 듯?

#### CMD
```dockerfile
CMD ["executable", "param1", "param2"]
CMD command param1 param2
CMD bundle exec ruby app.rb
```
- 도커 컨테이너가 실행되었을 때 실행되는 명령어를 정의한다.
	- 빌드할 때는 실행되지 않음
	- 여러 CMD가 있다면 마지막 CMD만 실행됨
		- 한꺼번에 여러 개의 프로그램을 실행하고 싶다면 `run.sh` 파일을 작성하여 데몬으로 실행하나, `supervisord`나 `forego` 같은 여러 프로그램을 실행하는 프로그램을 사용한다.

#### WORKDIR
```dockerfile
WORKDIR /path/to/workdir
```
- `RUN`, `CMD`, `ADD`, `COPY` 등이 이루어질 기본 디렉토리를 설정한다. 
- 이게 없다면 각 명령어의 현재 디렉토리는 매 줄마다 초기화된다. 
	- `RUN cd /path`를 입력하더라도 다음 명령어에서 초기화된다는 뜻

#### EXPOSE
```dockerfile
EXPOSE <port> [<port>...]

# ex
EXPOSE 4567
```
- 도커 컨테이너가 실행되었을 때 요청을 기다리는(`Listen`) 포트를 지정한다. 여러 개 설정 가능.
- 

#### VOLUME
```dockerfile
VOLUME ["/data"]
```
- 컨테이너 외부에 파일 시스템을 마운트할 때 사용한다. 
- 업데이트마다 내용물이 날아가는 컨테이너 특성 상, 지정하지 않아도 좋으나 기본적으론 지정하는 게 좋다.

#### ENV
```dockerfile
ENV <key> <value>
ENV <key>=<value> ...

# ex)
ENV DB_URL mysql
```
- 컨테이너에서 사용할 환경변수를 지정함.
- 컨테이너 실행 시 `-e` 옵션을 사용하면 기존 값을 오버라이딩하게 된다.

- 이외의 명령어는 [공식문서](https://docs.docker.com/engine/reference/builder/)참고


## build 분석

```
(1) Sending build context to Docker daemon  5.12 kB
```
- 빌드 명령어를 실행한 디렉토리의 파일들을 `빌드 컨텍스트Build Context`라고 한다.
- 이 파일들은 도커 서버(Daemon)로 전송한다. 도커 서버가 작업하기 위해선 미리 파일을 전송해야 한다.

```
(2) Step 1/10 : FROM ubuntu:16.04
```
- `Dockerfile`을 1줄씩 수행한다. 
- ubuntu:16.04 이미지를 다운받음

```
(3) ---> f49eec89601e
```
- 명령어 수행 결과는 이미지로 저장된다. 위 이미지는 `ubuntu:16.04` 이미지의 ID임

```
(4) ---> Running in f4de0c750abb
(5) ---> 4a400609ff73
```
- 바로 이전에 생성된 이미지 `f49eec89601e`를 기반으로, `f4de0c750abb` 컨테이너를 임시로 생성하여 실행한 다음 결과를 이미지로 저장한다.

```
(6) Removing intermediate container f4de0c750abb
```
- 명령어를 수행하기 위해 임시로 만들었던 컨테이너를 제거한다.

```
(8) Step 3/10 : RUN apt-get -y update
```
- 이후 위 과정(`이전 이미지 기반 임시 컨테이너 생성` -> `명령어 수행 후 결과 이미지로 저장` -> `임시 컨테이너 제거` -> `저장된 이미지로 다시 임시 컨테이너 생성 ->` ....)의 작업을 매 줄마다 반복한다.
- 명령어를 실행할 때마다 이미지 레이어 저장 & 다시 빌드할 때 Dockerfile이 바뀌지 않았다면 기존에 저장된 이미지를 캐시처럼 사용한다.

- 위 개념은 **최적화된 이미지 생성**을 만드는 데 있어 중요함

### 도커 이미지 리팩토링
- 위의 이미지는 최적화 이슈가 있다.

#### Base Image
- 예제의 이미지는 `ubuntu`가 베이스이나, 사실 훨씬 간단한 `ruby` 베이스 이미지가 존재한다. 
```bash
# before
FROM ubuntu:16.04
MAINTAINER subicura@subicura.com
RUN apt-get -y update
RUN apt-get -y install ruby
RUN gem install bundler

# after
FROM ruby:2.3
MAINTAINER subicura@subicura.com
```
- 세부적인 설정이 필요하지 않다면 이미 있는 베이스 이미지를 사용하는 것이 편리하다.

#### Build Cache
- 이전에 빌드했던 이미지를 다시 빌드하면 훨씬 빠르게 완료된다. **빌드 과정에서 각 단계가 이미지 레이어로 저장되고 다음 빌드에서 캐시로 사용되기 때문**

- 도커는 빌드할 때 Dockerfile의 명령어가 수정되었거나 추가하는 파일이변경될 때 캐시가 깨지고, 그 이후 작업은 새로 이미지를 만들게 된다.
- `ruby gem` 패키지 설치 과정은 꽤 시간이 오래 소요되므로, 최대한 캐시로 빌드 시간을 줄일 필요가 있다.
- 위 예제에서 캐시가 깨지는 부분은 이렇다.
```
COPY . /usr/src/app    # <- 소스파일이 변경되면 캐시가 깨짐
WORKDIR /usr/src/app
RUN bundle install     # 패키지를 추가하지 않았는데 또 인스톨하게 됨 ㅠㅠ
```
	- 복사하는 파일이 이전과 다르다면 캐시를 사용하지 않고 명령어는 다시 실행되게 된다.

- 이를 해결하기 위해, `ruby gem` 패키지를 관리하는 `Gemfile`은 잘 수정되지 않는다는 점을 이용해 순서를 바꿔 작성할 수 있다.
```
COPY Gemfile* /usr/src/app/ # Gemfile을 먼저 복사함
WORKDIR /usr/src/app
RUN bundle install          # 패키지 인스톨
COPY . /usr/src/app         # <- 소스가 바꼈을 때 캐시가 깨지는 시점 ^0^
```
- 소스 복사 이전으로 gem 설치하는 부분을 옮겼는데, 이 다음부터는 매번 `gem`을 설치하지 않아도 되기 때문에 더 빠르게 빌드할 수 있다.
- `패키지 매니저`와 비슷한 전략으로 사용하면 된단다.

#### 명령어 최적화
- 이미지 빌드 시 불필요한 로그는 무시하고 문서 파일도 생성할 필요가 없다
``` shell
# before
RUN apt-get -y update

# after
RUN apt-get -y -qq update
```
- `-qq` 옵션은 로그를 출력하지 않게 한다. 리눅스 명령어는 보통 `quite`에 관련한 옵션이 있으니 적절히 활용하자.

```shell
# before
RUN bundle install

# after
RUN bundle install --no-rdoc --no-ri
```
- `--no-rdoc`와 `--no-ri` 옵션으로 필요 없는 문서 생성을 막아 이미지 용량을 줄이고 빌드 속도를 빠르게 했다.

#### 비슷한 명령어 묶기

```bash
RUN apt-get -y -qq update
RUN apt-get -y -qq install ruby

# after
RUN apt-get -y -qq update && \
    apt-get -y -qq install ruby
```
- 도커 이미지는 스토리지 엔진에 따라 레이어 수가 127개로 제한되어 있는 경우도 있기 때문에 명령어의 수를 줄이는 게 좋다.

#### 최종
`Dockerfile`
```dockerfile
FROM ruby:2.3

MAINTAINER subicura@subicura.com

COPY Gemfile* /usr/src/app/

WORKDIR /usr/src/app/

RUN bundle install --no-rdoc --no-ri

COPY . /usr/src/app/

EXPOSE 4567

CMD bundle exec ruby app.rb -o 0.0.0.0
```

- 