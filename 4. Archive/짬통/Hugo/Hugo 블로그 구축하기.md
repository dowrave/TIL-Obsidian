#### 1. Git, Hugo 설치
- Hugo의 경우 내 노트북 기준 `amd64` 버전을 설치하며, `C:\Hugo\bin`에 설치된다. `exe` 파일이 있는데, 이를 사용하기 위해 환경 변수에서 지정도 해준다.

#### 2. github 레포지토리 생성
- 하나는 `myblog`
- 다른 하나는 `<닉네임>.github.io` 로 생성하며, (**꼭 Public**)
- 둘 모두 만들때 `add readme.md`를 켰음

#### 3. 프로젝트 만들기
```shell
hugo new site <프로젝트 이름>
```

#### 4. 프로젝트 깃 시작
```shell
cd <프로젝트 이름>
git init # 로컬 브랜치 master
```

#### 5. `myblog` 레포지토리에 연결
```shell
git remote add origin 
```

#### 6. 테마 서브모듈로 해서 다운받고, config.toml 파일 수정
```shell
git submodule add origin <테마 깃 레포지토리> themes/<테마이름>
```
- config.toml
```
baseURL = 'http://<유저닉네임>.github.io'
languageCode = 'ko-kr'
title = 'Twae's Blog'
theme = '<테마이름>'
```

#### 7. Public 폴더 삭제 및 아래 코드 입력
```shell
git submodule add -b main <static files repository URL> public
```

#### 8. 글 하나 생성
```shell
hugo new posts/my-first-post.md
```

#### 9. 서버 실행
```shell
hugo server -D
# 종료는 ctrl + c
```

#### 10. 종료 후 빌드(Public서버에 저장)
```sh
hugo -D
```

#### 11. 종료 후
1. `blog` 전체 push
```shell
git add .
git commit -m "<commit message>"
git push origin master
```

2. `public` 폴더 push
```shell
cd public
git add .
git commit -m "<Commit Message>"
git push origin main # 7에서 main으로 만들었기 때문
```


- **ChatGPT는 신이야!**