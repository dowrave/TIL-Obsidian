
## Localization 작업 계속
- 텍스트 영역 보고 레이아웃 만지고 이런 작업을 계속함

### 작업 내용

> [!note]
> - 정예화 패널 키 상단 `const`로 관리
> - 오퍼레이터 인벤토리 패널 이름값 텍스트 크기 자동화 & 레이아웃 수정
> - `NotificationToast`에 들어가는 텍스트들 string -> 키값 기반으로 변경
> - `OperatorSlot, NotificationToast` 텍스트 크기 조정

#### 정예화 패널 키값들 상단에 `const`로 관리
- `const`의 사용 조건 & 의미 : **런타임에 바뀌지 않는다, 바뀌면 안된다**, 프로그램 전체에서 동일한 의미를 가진다 
```cs
private const string AdditionalSkillString = "UI_PROMOTION_ADDITIONAL_SKILL";
private const string CannotPromoteBase = "UI_PROMOTION_CANNOT_BASE";
private const string CannotPromoteLevel = "UI_PROMOTION_CANNOT_LEVEL";
private const string CannotPromoteItem = "UI_PROMOTION_CANNOT_ITEM";
```
> - `Localize String Event`처럼 해당 키-값의 레퍼런스를 참조하는 게 인스펙터에서는 가능한데, 코드 단위에서는 연결 방법이 딱히 보이진 않음(테이블의 키값이 바뀔 때 코드를 바꿀 필요 없이 자동으로 바뀌는 방법을 얘기함). 찾아보면 방법이 나올 것 같긴 한데 일단 이렇게만 처리함
> - **상수의 네이밍 컨벤션은 C#에서는 파스칼 케이스**가 일반적
> 	- 각 명칭은 이렇다고 함
> 		- `PascalCase` ( C# )
> 		- `SCREAMING_SNAKE_CASE`  (C / Java)
> 			- 시각적으로 절대 안 바뀐다는 것을 강조함. C#은 그 정보가 이미 `const`에 들어있으니 굳이 중복 표현하지 않는다는 철학.
> - C#의 상수는 암묵적으로 `정적`임. (그래서 파스칼 케이스를 쓰는 것도 있나?)


#### 오퍼레이터 인벤토리 패널 이름값 텍스트 사이즈 자동화
- 일본어의 경우 영역을 벗어나려 해서 수정
- 하는 김에 레이아웃도 다듬음(정렬, 높이값, 하이어라키 등등)


#### NotificationToast에 들어가는 텍스트 string -> 키값 기반으로 변경
- 코드 레벨에서 키값 넣는 식으로 구현함
- `Smart String` 부분도 이런 식으로 집어넣으니 잘 동작함

![[Pasted image 20260528164257.png]]
```cs
string message = LocalizationSettings.StringDatabase.GetLocalizedString(
	LocalizationManager.TableMessage, 
	SetDefaultSkill,
	arguments: new { value = currentSelectedSkill.SkillName }
);

NotificationToastManager.Instance.ShowNotification(message);
```

- 이 과정에서 `NotificationToastManager`도 아래처럼 공통 로직 + 세부 파라미터를 받는 식으로 수정
```cs
private NotificationToast PrepareToast()
{
	if (notificationToastPrefab == null)
	{
		Logger.LogError("NotificationToast 프리팹이 할당되지 않음");
		return null;
	}

	if (activeToasts.Count >= maxVisibleNotifications)
	{
		// 가장 오래된 토스트는 리스트의 맨 앞 요소
		NotificationToast oldestToast = activeToasts[0];
		activeToasts.RemoveAt(0);

		// Dismiss를 호출하면 onToastClosed 콜백이 실행, 리스트에서 자동으로 제거된다. 
		oldestToast.Dismiss();

		UpdateToastPositions();
	}

	GameObject toastObject = Instantiate(notificationToastPrefab, notificationContainer);
	NotificationToast newToast = toastObject.GetComponent<NotificationToast>();

	// 새로운 토스트 추가
	activeToasts.Add(newToast);

	return newToast;
}

// 사용 방법 1
public void ShowNotification(string message)

// 사용 방법 2
public void ShowNotificationWithKey(string table, string key)
```
> - 근데 높이 계산하는 게 비뚤어져서 그거 체크해봄
> 	- 토스트를 생성하고 나서 높이 계산을 했기 때문에 첫 토스트의 위치가 이상하게 잡혔다. 토스트 생성 전에 높이 계산하면 됨. 코드는 안 넣음

- `NotificationToast`의 `RaycastTarget` 끔

> 이외에도 광클하면 마지막 토스트의 위치가 맨 밑으로 가는 현상이 있음
> 근데 이걸 수정할 필요성은 못 느끼므로 넘어감

- 영어, 일본어 버전의 텍스트 넣음

- 나머지 `NotificationToast`에 string 들어가는 부분들 수정 완료

#### OperatorSlot, NotificationToast 텍스트 크기 조정
- `OperatorSlot` 완료, 좌우 패딩 추가
- `NotificationToast` 작업 완료


### 발견한 이슈 / 고칠 필요성 느끼는 부분
>[!note]
>- 버튼 1번째 클릭 시 동작하지 않음
>- 정예화 패널 공격거리 증가 영역 텍스트의 오른쪽으로 밀기

- 버튼 1번쨰 클릭 때 동작 안하는 현상
