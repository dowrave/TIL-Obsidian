- 현재 어떤 작업 중인지 기록 중
# 참고
- **옵시디언으로 봐야 제대로 보인다.**
	- **옵시디언으로 작성된 만큼 깃허브의 마크다운에서는 지원하지 않는 기능들이 있을 수 있다.** `[[]]`, 이미지 첨부 방식 등이 대표적.
- `[[]]` 링크는 `유니티/보관함`이나 `작업 일지/직접 작성/일지`에 대부분 있다.

# 작업 일지
## 짭명방 
- 지난 내역 : [짭명방 프로젝트 일지 링크](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EC%9C%A0%EB%8B%88%ED%8B%B0%20-%20%EC%9E%91%EC%9D%80%20%EB%AA%85%EB%B0%A9%20%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/%EC%9E%91%EC%97%85%20%EC%9D%BC%EC%A7%80/%EC%A7%81%EC%A0%91%20%EC%9E%91%EC%84%B1)
- [[기타 참고 사항]]

>[!issue]
> 간헐적인 이슈들 : 계속 발생하는 경우에는 수정하지만 아니라면 남겨둠
> - 적이 이미 사라졌는데 계속 해당 위치를 공격하는 현상
> - `ArcaneFieldSkill` : 스킬을 썼음에도 효과가 제대로 적용되지 않는 현상
> 	- 위치가 애매하게 걸쳐지는 경우가 있나? 의심은 있는데 상황을 재현하기 어려움
> - `Artillery`의 공격 효과음이 제대로 나오지 않는 현상

>[!note]
>- 할 일들 
>1. `TitleScene`이랑 `GameManagement`, `GameSession` 등을 구현하면서 게임의 전체적인 로직이 틀어지는 현상이 있다. 일단 전반적으로 데이터 관리 흐름을 점검함.
>2. `TitleScene` 완성하기
>	- 도형 완성
>	- 게임 시작 시 연출 (도형이 멀어지면서 로딩게이지 양 끝에서 가운데로 모이는 현상, 명방의 그것)
>- 2까지 한 다음 프로젝트 일시정지하고 자료구조 책 다 공부한 다음 엔딩 크레딧 정도만 만들고 끝낼 예정이다. 


>[!note]
>- 마지막 맵 테스트 전 추가로 구현하고 싶은 것들
>- `BossBGM`의 보스 스폰 후의 BGM으로 전환되더라도 크게 바뀌었는지 모르겠다. 이 부분은 수정이 필요해보임.
>- `LightningHit`의 소리가 겹칠 때 너무 커진다. 갯수는 2개로 제한했는데도 그런데, 이전처럼 관리하고 제어하는 방법밖에 없나?

## 최근 5일

### 260817
- [[자료구조 - 5. 스택]]

### 260815
- 책 목차 + Claude로 공부한 내용.
- [[Effective Csharp - 1. Csharp 언어 요소]]
- [[Effective Csharp - 2. .NET 리소스 관리]]
- [[Effective Csharp - 3. 제네릭 활용]]
- [[Effective Csharp - 4. LINQ 활용]]
- [[Effective Csharp - 5. 예외 처리]]
- [[Effective CSharp - 유니티 관점의 요약]]
- [[Csharp - 클로저(Closure)]]
	- 클로저 예제 틀리는 거 보면 Claude를 믿어도 되나? 싶긴 했다. 대체로 맞겠지...
### 260814
- [[짭명방_260814]]
>[!done]
>- `TitleScene` 도형 구조 수정
>- `TitleScene` 구현
>	- 구현 중 이슈 수정
>		- 최초 씬이 옮겨지면서 `TutorialManager`의 동작도 수정되어야 함
>		- `SceneLoader` 메서드들 수정
>		- `GameManagement`(게임 전체) / `GameSession`(세이브 파일 정보) 로 `DontDestroyOnLoad` 요소들 구분
>		- `PlayerDataManager`을 `GameDataManager(GameManagement)`와 `PlayerSessionDataManager(GameSession)`으로 구분
>	- `TitleScene`부터 시작하는 흐름 수정 중

### 260813
- [[짭명방_260813]]
>[!done]
>- 어제의 이슈
>	- `Operator` : 저지 로직이 제대로 동작하지 않는 현상
>- 추가 이슈
>	- `Enemy` 공격 시 `AttackType = None`으로 나타나는 현상
> - StageScene 구현
> 	- 버텍스에서 퍼져나가는 동심원이 게임 뷰에서 보이지 않는 현상 수정
### 260812
- [[짭명방_260812]]
>[!done]
>1. 이전에 못 끝낸 거
>	- `GetActionableGridPos`가 제대로 들어오지 않는 현상
>		- 내부의 메서드 / 프로퍼티의 역할이  애매했던 부분도 수정
>	- 배치가 제대로 되지 않는 현상
>2. 추가 이슈
>	- `IsTargetInRange` 동작 이슈
### 260810
- [[짭명방_260810]]
>[!done]
>-  기타 수정
>	- 프로퍼티와 백킹 필드 사용에 관해
### 260806
- [[짭명방_260806]]
>[!done]
>1. AttackType, AttackRangeType 필드 위치 - `Status.Stat`으로 설정
>2. `Enemy` 컨트롤러 세부 구현
>3. 테스트 중 - 기타 이슈 발생 중
### 260805
- [[짭명방_260805]]
>[!done]
>- 또팩토링 : `UnitEntity`의 컨트롤러들 구조 고치기 (유사 파사드 패턴)
>	- `UnitEntity` : `_buff, _health, _stat`을 `UnitStatusController`인 `_status`의 아래로 묶었음
>	- `DeployableUnitEntity` : `_deployable` 1개만 있으므로 현상태 유지
>	- `Operator` : `_action, _block, _skill`을 `OpCombatController`인 `_combat`의 아래로 묶음
>- 막힌 지점
> 	- `AttackType`, `AttackRangeType`의 관리

### 최근 작업 내용 - 블로그
- 마지막 수정 내역 : [[블로그_260723 - 불편한 것들 해결]]

>[!done]
>- 프론트엔드
>	- 게시판 : 글쓰기 버튼 상단에도 추가
>	- 글쓰기 : `list` 제거
>		- 복붙할 때 불편해서 아예 빼버렸음
>	- CSS : `ql-indent` 시리즈의 회색으로 작게 글씨를 변경한 부분들 제거
>- 백엔드
>	- 페이지네이션이 정상적으로 적용되지 않고 모든 글을 가져오는 현상 수정
## 짭명방
- [짭명방 프로젝트 일지 깃허브 링크(프로젝트 자체는 Private 전환)](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EC%9C%A0%EB%8B%88%ED%8B%B0%20-%20%EC%9E%91%EC%9D%80%20%EB%AA%85%EB%B0%A9%20%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/%EC%9E%91%EC%97%85%20%EC%9D%BC%EC%A7%80/%EC%A7%81%EC%A0%91%20%EC%9E%91%EC%84%B1)
## 블로그
- [React + Django 프로젝트 일지 월별 작업 기록 깃허브 링크](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EB%B8%94%EB%A1%9C%EA%B7%B8%20%EB%A7%8C%EB%93%A4%EA%B8%B0/%EC%9B%94%EB%B3%84%20%EC%9E%91%EC%97%85%20%EA%B8%B0%EB%A1%9D)
