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

>[!TODO]
>- 260831 기준
>- 미리보기 메타 데이터 : 데이터가 없는 데 있다고 처리되는 경우가 있는 듯? **"현재 해당 슬롯에 데이터가 있는가"에 관한 로직부터 다시 점검하는 걸로 시작하면 될 것 같다.** 

## 최근 5일

### 260831
- [[짭명방_260831]]
>[!done]
>- `GlobalPopupManager / PersistantCanvas` 구현
>	- 여러 씬에서 공통으로 사용할 팝업을 관리
>	- `NotificationToastManager`도 `ToastManager`로 이름 변경 후 `GameManagement`로 이사감
>- `SaveSlotLayout` 작업하기
>	- 삭제, 시작, 뒤로 가기 버튼 구현
>	- (TODO) 세부적인 동작은 구현 필요
### 260829
- [[짭명방_260829]]
>[!done]
>- 기타 이슈
>	- 이미지 비율 깨지는 현상 : [[유니티 - 9슬라이싱 스프라이트]]
>	- 공통 스타일 관리 방법...은 일단 SO만 만들어두고 이 데이터를 참조로 눈으로 색을 집어넣는 방식으로 구현

>[!wip]
>- `TitleLayout` 만들기
>	- 시작 / 계속하기에 따른 기능 분리

### 260828
>[!wip]
>- 세이브, 로드 시스템 구현
>	1. `SaveSlot`의 구체적인 형태 구현
>	2. `TitleManager`와 `SaveSlotManager`의 기능 일부를 `MainLayout`과 `SaveSlotLayout`으로 분리
>		- `Layout`에서 `Manager`에 요청을 보내서 정보를 읽어오는 방식
>- 일반적인 유니티의 코드 컨벤션을 정리하려다가 말았다. 

### 260826
- [[짭명방_260826]]
>[!done]
>1. SaveSlotPanel 구현
>2. 기타 수정 사항과 질문/답변
>	- `SaveSlotManager`의 위치 수정
>	- 메서드를 어떻게 작성해야 하는가?
>		- 토글 메서드
>		- 기능에 따른 메서드 분리
### 260825
- [[짭명방_260825]]
>[!done]
>1. 데이터 저장 방식 `PlayerPrefs`에서 `Application.persistantDataPath`로 변경, 옵션 관련 설정을 제외한 데이터 저장 방식을 전부 변경.

## 짭명방
- [짭명방 프로젝트 일지 깃허브 링크(프로젝트 자체는 Private 전환)](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EC%9C%A0%EB%8B%88%ED%8B%B0%20-%20%EC%9E%91%EC%9D%80%20%EB%AA%85%EB%B0%A9%20%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/%EC%9E%91%EC%97%85%20%EC%9D%BC%EC%A7%80/%EC%A7%81%EC%A0%91%20%EC%9E%91%EC%84%B1)
## 블로그
- [React + Django 프로젝트 일지 월별 작업 기록 깃허브 링크](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EB%B8%94%EB%A1%9C%EA%B7%B8%20%EB%A7%8C%EB%93%A4%EA%B8%B0/%EC%9B%94%EB%B3%84%20%EC%9E%91%EC%97%85%20%EA%B8%B0%EB%A1%9D)
