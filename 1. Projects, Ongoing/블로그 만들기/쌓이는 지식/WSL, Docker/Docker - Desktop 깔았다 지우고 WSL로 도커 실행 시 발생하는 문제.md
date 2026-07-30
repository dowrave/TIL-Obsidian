- 이슈 상황은 제목 그 자체임
- 즉 평소에 WSL 2로 도커를 쓰다가, 데탑에서 써야징~ 하고 데탑을 깔았다가, 다시 지운 상태에서 우분투에서 도커를 실행하고 똑같은 컴포넌트를 빌드하는 과정에서 이런 로그가 떴다.
```sh
failed to solve: node:20: failed to resolve source metadata for docker.io/library/node:20: error getting credentials - err: exec: "docker-credential-desktop.exe": executable file not found in $PATH, out: ``
```

- 이걸 ChatGPT님께 여쭤보니, `~/.docker/config.json` 파일에서 아래 부분을 수정하란다.
```json
{
  "auths": {
    ...
  },
  "credsStore": "" 
}
```
> Docker for Desktop을 깔았다가 지운 상태라면, `credsStore`에 값이 들어가 있다. **그 값을 위처럼 빈칸으로 바꾸거나, 키-밸류 전체를 지우면 됨.**

- 그런데 어디에 있는 `config.json` 파일이냐?
- 내 경우
	1. 윈도우에 있는 `%USERPROFILE%\.docker\config.json` 경로를 수정했음 -> 해결 안됨
	2. 우분투에서도 해당 파일이 있다. 아래처럼 접근한다.
```sh
cd ~ # 홈 디렉토리로 이동
nano .docker/config.json # config.json 파일에 접근
```
> 수정은 위처럼 똑같이 하면 된다.
> `nano` 에디터의 경우, `ctrl + x`로 탈출이 가능하며, 탈출 과정에서 저장 여부를 물어본다.