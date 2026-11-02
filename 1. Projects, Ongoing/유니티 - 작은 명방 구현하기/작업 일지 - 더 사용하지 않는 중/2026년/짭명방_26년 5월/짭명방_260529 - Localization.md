

## 작업 중 키값 관리 구조에 대한 생각

>[!note]
>- 한 줄 요약 : `LocalizationManager`에서 스크립트 단위에서 쓰이는 `StringTable`의 키값을 관리하기로 함

- 지금은 개별 UI 요소에서 필요한 `키 string`를 그때그떄 **테이블에 있는 걸 옮겨서 적는 식으로 작업**하고 있음
- 근데 테이블의 키값이 바뀌는 경우가 발생했음. 

### 발생한 상황
- 요약 : 키값을 고쳐야 할 상황이 발생함. 지금 방법으론 시간이 오래 걸릴 듯

1. `MESSAGE_RESET_GROWTH`라는 키가 있었음. 토스트에서 "모든 오퍼레이터의 레벨 초기화 및 육성 재화 회수 완료"라는 메시지를 띄우는 키임
2. 근데 그걸 확인 팝업에서 "모든 오퍼레이터의 육성 상태를 초기화합니다. 진행하시겠습니까?"라는 새로운 키를 만들어야 함
3. 이 둘을 구분할 필요가 생겼고, 하나는 `MESSAGE_ASK_RESET_GROWTH`, 다른 하나는 `MESSAGE_COMPLETE_RESET_GROWTH`로 관리하기로 함. 

그런데 `COMPLETE`가 기존의 키여서, 이걸 수정하기 위해서는 해당 팝업을 띄우는 코드로 접근해서 그 String 값을 수정해야 함. 다른 경우에도 **각각의 컴포넌트에 접근해서 문자열을 찾아서 수정하는 방식이 번거로울 것 같음.**

### 해결책 - 상수 클래스
- 클로드가 준 답변은 **키값들을 관리하는 상수 클래스**를 만들라는 것
- `LocalizationManager`를 쓰기로 함(이미 테이블에 대한 상수 클래스 역할을 하고 있음)

>[!note]
> **`StringTable`에 키값을 추가할 때, 상수 클래스에도 해당 키값을 갖는 상수 변수를 하나 만들라는 것이다.**
- 이중작업일 수 있지만, 여기저기 스크립트에 접근하는 것보다 "이걸 수정하려면 여기에 접근하면 된다"라는 규칙이 있기 때문에 나중에 관리할 때 훨씬 편해짐

#### 기타 고려사항 1. 모든 키값을 넣어야 하는가?
그러면 모든 키값을 넣어둬야 할까? 
1. 스크립트 단위에서 쓰이지 않더라도 키값을 추가할 때마다 넣는다
2. **스크립트 단위에서 쓰일 때만 키값을 넣는다.**

2번을 권장한다. 
- 2번처럼 작업하면 **키값이 상수 클래스에 있다는 것 자체가 "스크립트에서 쓰이고 있다"** 를 보장함
- 1번처럼 작업하면 
	- 기획 / 번역 단계에서 키값을 추가할 때마다 개발자가 상수 클래스를 동기화해야 함
	- 실제로 쓰이지 않는 상수가 발생, 키가 쓰이는지 여부에 대한 판단이 어려움

언젠가 쓸지 모르니까 넣는 건 관리 비용만 올라가고 신뢰도는 내려간다. 

#### 기타 고려사항 2. 키값 네이밍 규칙
- 키값은 `SCREAMING_SNAKE_CASE`를 쓰는 게 표준이다.
	- 데이터 개념임
	- 기획자 / 번역가도 보는 값임.
	- `대문자 + 언더스코어`는 "이건 코드가 아니라 식별자"라는 정보를 직관적으로 전달함

- `PascalCase`는 개발자(프로그래머)들 사이에서만 쓴다고 생각해도 될 듯.
	- 어제도 다뤘지만 상수는 파스칼 케이스가 표준이란다.

#### 기타 고려사항 3. 구조
- 현재 `LocalizationManager`에 테이블과 키를 상수 변수로 모두 넣어두고 있음
- 근데 키가 늘어날수록 테이블 소속 파악이 어려워지므로 구조화를 할 필요가 있음

- 구조화 방법은 크게 2가지임 : 구현 방식, **의미 기준**
```CS
// 테이블명/키 분리 구조 → 둘이 연결되어 있다는 게 코드에서 안 보임
GetLocalizedString(LocaleKeys.TableMessage, LocaleKeys.KeyAskResetGrowth);

// 기능별 중첩 구조 → "Message 테이블의 AskResetGrowth 키" 가 한눈에 보임
GetLocalizedString(LocaleKeys.Message.Table, LocaleKeys.Message.AskResetGrowth);
```
> 의미로 나누는 게 읽기 훨씬 편함

- 최종적으로는 이런 느낌이 됨
```CS
public static class Operator
{
	public const string Table = "Operator";
	public const string SwordName = "OPERATOR_NAME_SWORD";
	public const string SwordDesc = "OPERATOR_DESC_SWORD";
}

public static class Enemy
{
	public const string Table = "Enemy";
	public const string GoblinName = "ENEMY_NAME_GOBLIN";
	public const string GoblinDesc = "ENEMY_DESC_GOBLIN";
}
```
> 스크립트 단위에서 **참조(?) 테이블을 하나 만든다...라고 생각하면 이해가 빠를 듯**.

## Localization 적용된 요소들

### ConfirmationPopup
- 들어갈 문구는 개별 스크립트에서
- 아래의 확인 / 취소 버튼
- 추가 : `blurArea`가 활성화되지 않는 경우가 있어서 일괄적으로 활성화되게 함
### OpRoleInfoPopup
![[Pasted image 20260529180925.png]]
> 각 요소를 테이블에서 어떻게 분리해야 할까?
> - 캐릭터마다 다른 건 `Operator`로
> - 텍스트가 동일하다면 `UI`로

- 처음에 죄다 UI에 때려박았는데 넣고 보니까 좀 이상해보인다.
- 오퍼레이터의 디테일한 설명은 `Operator` 테이블에서 관리

- 영어 버전
![[Pasted image 20260529182537.png]]

- 일본어 버전
![[Pasted image 20260529182558.png]]

### 튜토리얼 전체
- 지금 구현되어 있는 내용에 맞춰서 키값에 넘버링을 달면 됨
- 아래 느낌으로 작업
	- `TUTORIAL_SQUAD_EDIT_01`
	- `TUTORIAL_STAGE_01`
	- `TUTORIAL_MAIN_MENU_01`

- 이를 위해 `TutorialData` 내부에 있는 `TutorialStep`에 `List<string> keys` 필드를 추가하고 맞춰나가면 된다.

- **미리 유지보수를 어떻게 할지까지 생각하진 말 것**, 지금 있는 걸 명확하게 표현할 것

- 그럼에도 이런 상황은 나오더라
>[!question]
>- 어떤 문장이 반복되는 경우는 어떻게 구현해야 할까?
>- 예를 들어서 "빈 슬롯을 클릭하시오" 같은 문구는 여러 번 나옴

> **맥락이 달라지면 내용이 달라질 여지가 있기 때문에 다른 키값에 같은 내용을 넣어두는 게 좋다.** 순서가 들어가는 상황은 중복을 허용해주는 게 좋음.
> 1. 맥락이 다르면 텍스트가 달라질 수 있다. 동일한 문구지만 단계마다 미묘하게 다른 표현이 필요해지는 경우가 있다. 나중에 수정이 필요해질 경우에 키값까지 고쳐야 하는 상황이 옴
> 2. 순서가 의미를 가지는 경우는 내용이 동일하더라도 그 순서를 보여주는 편이 더 좋음
> 3. 공통 키를 재사용하는 건 맥락과 무관하게 항상 동일해야 하는 경우로, 지금 같은 경우는 맥락 상 동일한 상황이 나온 것일 뿐임.

- 코드 단위에서는 전체 텍스트를 가져오는 부분만 수정하면 됨
```cs
// fullText = step.dialogues[currentPageIndex];

fullText = LocalizationSettings.StringDatabase.GetLocalizedString(
	LocalizationManager.Tutorial.TableName,
	step.keys[currentPageIndex]
);
```

- 기존 `Dialogue`에 있던 텍스트값들을 `Tutorial` `String Table`로 옮기고 `TutorialData`의 `Keys`에는 테이블의 키값을 집어넣는 작업을 수행함
	- 대충 30개줄 정도 되나?
	- 다 옮겼다. 이제 `Dialogue` 필드를 지우고 테스트
	- `SO`에 분명히 키값을 넣었는데 안 들어간 경우도 있다. 이런거 수동으로 저장되게 좀 해주면 안되나?

- 일단 오늘은 `TutorialData`에 해당하는 내용들의 번역본만 넣어놓고 마무리함

>[!note]
>- 현재 눈에 띄는 이슈 (1번은 반드시 처리해야 함)
>1. `Stage` 튜토리얼이 정상적으로 진행되지 않음
>2. 버튼 2번씩 클릭되는 문제

