[원문](https://chanhuiseok.github.io/posts/git-3/)
- 오픈소스에 기여하는 방법


1. 기여하려는 저장소에서 Fork하기
	- `Fork` : 해당 레포지토리의 내용을 내 레포지토리로 가져오는 것
![[Pasted image 20230103171430.png]]
	- 내 레포지토리에서 내용 확인
		- `forked from 원문` 이 있으면 잘 된 거

2. 내 컴퓨터에 저장소 Clone

3. 원격 저장소 Remote 설정하기
	- 내가 PR을 보낼 곳을 추가함 (`remote_repo_name`은 그냥 내가 임의로 지정하는 건가봄?)
	- 이 떄 원격 저장소의 git 주소는 fork 하기 전의 원래 저장소를 의미한다
	- 이걸 하는 이유는 fork 저장소를 원격 저장소(원본)의 최신 커밋으로 내용을 변경해야 하기 때문에 진행한다.
```sh
git remote add {remote_repo_name} {origin_repo}.git
```

4. PR용 Branch 생성하기
	- `clone`해온 처음 브랜치는 기본 `master`이다. 여기에 **코드를 수정하고 PR을 보낼 용도로 사용할 새로운 Branch를 생성**해야 한다.
```sh
git checkout -b {branch_name}
```
- 위 명령어로 **Branch가 만들어짐 + 현재 Branch가 새로 만든 것으로 변경**됨

5. 코드 수정하기
- 오픈소스에서 기여할 부분을 수정함
```sh
git add .
git commit -m "커밋 메시지는 프로젝트의 규칙이 있다면 그것을 따를 것"
```

6. **PR용 branch**에 푸시하기
- PR용 Branch = 4.에서 생성한 그거
```sh
git push origin {4.에서 생성한 브랜치 이름}
```

7. Fork했던 원본 깃허브 사이트에 들어가기
![[Pasted image 20230103172209.png]]
- `Compare & Pull Request` 버튼이 활성화된 걸 볼 수 있음

![[Pasted image 20230103172236.png]]
- 프로젝트에서 미리 설정된 `PR(Pull Request)` 양식이 뜬다. 여기의 양식과 규칙에 맞게 기여한 것들을 작성하면 됨
- 작성 후 `Create Pull Request` 버튼을 눌러서 오픈소스 프로젝트에서 내 PR이 승인되어 Merge되는 걸 기다림

8. PR 승인이 되었다면 Branch 삭제하기(이거 맞나)
- 메인 저장소에 Merge되었다면, 4.에서 생성한 PR용 브랜치는 작업이 끝났기 때문에 삭제해도 됨
```sh
# local branch 삭제 (현재 브랜치인 경우 삭제 안됨)
git branch -D BRANCH_NAME


# 둘 중 하나 : remote branch 삭제
git push origin :BRANCH_NAME
git push origin --delete BRANCH_NAME
```

- 로컬 브랜치는 `git branch`로 확인 가능
- 리모트 브랜치는 `git branch -r`로 확인 가능