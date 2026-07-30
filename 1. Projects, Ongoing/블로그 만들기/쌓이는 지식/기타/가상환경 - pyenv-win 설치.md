- `pyenv` : 별도의 파이썬들을 설치하지 않고, 그때그때 버전을 바꿔서 세팅할수 있게 도와준다.
- `pyenv-win` : `pyenv`는 윈도우에는 없음. 이를 윈도우에 맞게끔 포팅한 것.
- [깃허브 레포지토리](https://github.com/pyenv-win/pyenv-win/blob/master/docs/installation.md)

1. 깃허브에서 클론해옴
```sh
git clone https://github.com/pyenv-win/pyenv-win.git "$HOME\.pyenv"
```

2.  `시스템 속성 - 환경 변수` 에서
	1) `시스템 변수`에 `PYENV, PYENV_HOME, PYENV_ROOT`을 각각 추가한다. 값은 모두 `%USERPROFILE%\.pyenv\pyenv-win\` 으로 통일한다.
	2) `Path`에 `$USERPROFILE%\.pyenv\pyenv-win\bin`, `$USERPROFILE%\.pyenv\pyenv-win\shims` 을 추가한다.

3. 새로운 쉘을 실행, `pyenv --version`을 입력해 설치된 `pyenv`의 버전이 뜨는지 확인한다.

> 내 경우 새로운 쉘을 실행해도 실행 불가능한 명령어라 떠서, 재부팅한 다음 시도해봤다.  
> 그랬더니 `'chcp'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는 배치 파일이 아닙니다. 'cscript'은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램, 또는 배치 파일이 아닙니다`가 우르르 떴는데, 이는 **쉘을 관리자 권한으로 실행**하니 뜨지 않았다.


