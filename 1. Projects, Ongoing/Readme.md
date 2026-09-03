- 현재 어떤 작업 중인지 기록 중
# 참고
- **옵시디언으로 봐야 제대로 보인다.**
	- **옵시디언으로 작성된 만큼 깃허브의 마크다운에서는 지원하지 않는 기능들이 있을 수 있다.** `[[]]`, 이미지 첨부 방식 등이 대표적.
- `[[]]` 링크는 `유니티/보관함`이나 `작업 일지/직접 작성/일지`에 대부분 있다.

# 작업 일지
## 짭명방 
- 지난 내역 : [짭명방 프로젝트 일지 링크](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EC%9C%A0%EB%8B%88%ED%8B%B0%20-%20%EC%9E%91%EC%9D%80%20%EB%AA%85%EB%B0%A9%20%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/%EC%9E%91%EC%97%85%20%EC%9D%BC%EC%A7%80/%EC%A7%81%EC%A0%91%20%EC%9E%91%EC%84%B1)
- [[기타 참고 사항]]
- [[짭명방 - 이슈 정리]]

>[!note]
>- 할 일
>1. `TitleScene`이랑 `GameManagement`, `GameSession` 등을 구현하면서 게임의 전체적인 로직이 틀어지는 현상이 있다. 일단 전반적으로 데이터 관리 흐름을 점검함.
>2. `TitleScene` 완성하기
>	- 도형 완성 (디테일 필요)
>	- 게임 시작 시 연출 (도형이 멀어지면서 로딩게이지 양 끝에서 가운데로 모이는 현상, 명방의 그것)
>- 엔딩 크레딧 관련


## 최근 기록

### 260902
- [[짭명방_260902]]
>[!done]
>- `SaveSlotLayout` 요소들 정리
>- 기타 수정
>	- `OptionPopup` : `Title`로 돌아가기 버튼 추가
>- 이슈 수정
>	- `SaveSlotLayout`
>		- 빈 슬롯 여부 관리
>	- `MainMenuScene` 진입 시 클릭이 동작하지 않는 현상
>		- 개별 씬에 `EventSystem` 구현해놓는 걸로 복구

- 타이틀로 돌아가기 관련
	- 현재까지의 진행 상황을 저장하고 돌아가기 외에, 저장하지 않고 돌아가는 기능을 추가해야 하나?
	- 근데 **한 세션에 여러 개의 세이브 슬롯을 갖는 게임이 아니라서 굳이 필요없어보이긴 한다.**

## 짭명방
- [짭명방 프로젝트 일지 깃허브 링크(프로젝트 자체는 Private 전환)](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EC%9C%A0%EB%8B%88%ED%8B%B0%20-%20%EC%9E%91%EC%9D%80%20%EB%AA%85%EB%B0%A9%20%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/%EC%9E%91%EC%97%85%20%EC%9D%BC%EC%A7%80/%EC%A7%81%EC%A0%91%20%EC%9E%91%EC%84%B1)
## 블로그
- [React + Django 프로젝트 일지 월별 작업 기록 깃허브 링크](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EB%B8%94%EB%A1%9C%EA%B7%B8%20%EB%A7%8C%EB%93%A4%EA%B8%B0/%EC%9B%94%EB%B3%84%20%EC%9E%91%EC%97%85%20%EA%B8%B0%EB%A1%9D)
