
>[!done]
>- `Localization` 작업
>	- `Item` 관련 요소들 : `ItemData`, `ItemUIElement(+Small)`, `ItemInfoPopup`
>- 아이템 데이터 관리, SO ID 명명법 등도 수정했음

## `Localization` 작업 계속

### 작업 완료
- `ItemUIElement`, `ItemUIElement(Small)` UI 요소들 작업 완료
- 아래는 텍스트로 들어가되 데이터에 따라 변하는 값들에 대한 작업
#### ItemData
- 이 부분은 새로운 테이블을 만들어서 작업함. `Item` 테이블.
- 기존 필드 : `ItemName`, `Description`
- 수정 필드 : `_id` 
	- 기존에 `ItemName`을 `id`처럼 썼기 때문에 이걸 대체할 요소가 필요함
	- 현재 언어 상태에 영향을 받으면 안 됨
	- 이 **`_id`를 기반으로 `Name`, `Desc`에 관한 생성 규칙을 만듦**

>[!note]
>- 이 ID를 어떤 규칙으로 넣는 게 좋을까?에 대한 고민을 했다.
>1. `ITEM_{고유의 이름}`
>2. `{고유의 이름}`
>- 클로드의 답 : **1번. 키가 전역적으로 읽히기 때문**이다. 
>- 코드로 사용할 경우는 테이블까지 명시하긴 하지만.. 만들 때는 어떻게 쓰일지 모르니, 이렇게 만드는 게 좋겠다.

```cs
// ItemData.cs
[SerializeField] private string _ID = string.Empty;

public string NameKey => $"{_ID}_NAME";
public string DescKey => $"{_ID}_DESC";

public string GetName() => LocalizationSettings.StringDatabase.GetLocalizedString(LocalizationManager.TableName, key)

// LocalizationManager에 전역 string 필드로 테이블의 이름들을 관리하는 중임. 개별 SO에서 테이블 이름을 일일이 저장할 필요 없다.
```

#### UI 반영
- 아이템 데이터가 나타나는 부분들에 아래 코드를 집어넣으면 됨. 여기엔 참고 또 하라고 넣어둔다.
```cs
// UI에서 반영하는 방법
LocalizationSettings.SelectedLocaleChanged += OnLocaleChanged;

void OnLocaleChanged(Locale locale) => Refresh();

void Refresh()
{
	// 바꾸고 싶은 `TMP.text`의 내용을 SO에서 긁어와서 수정
}

// 대충 어디서 구독 해제
```

### ID 명명법 정리
- 모두 각자가 속한 `테이블_{고유의 이름}`을 대문자로만 작성
- 키값들에 관한 부분도 모두 정리했음


### 이슈 발견
- `TestManager`에서 스테이지 클리어 처리는 되는데 아이템 지급 코드가 정상 동작하지 않고 있음. 
	- 위에꺼 반영하려면 이것까지 체크해서 진행해야 하므로 이 버그부터 해결 진행함
- **아.. `StageData`에 있던 보상 아이템들이 날아갔음 ㅋ...**
	- 스테이지 완료와 보상 기준
		- **정예화를 2/3/3으로 맞추려고 했었음**
		- 25년 4월 작업본에 이런 내용이 있었다. 
			- 0정예화 1레벨 -> 50레벨에 필요한 경험치가 24900
		- **1-0의 경험치량은 50000, 1-1, 1-2의 경험치량은 75000으로 구성하면 될 듯**
		- 기본 클리어 보상은 알아서 설정. 
		- 양쪽 모두 갯수는 4의 배수만 맞춰주면 된다.

- [[기타 참고 사항#스테이지별 경험치 아이템 갯수]]에 아예 정리해둠. 

#### Localization에서 SO의 값 참조하게 하기 - Smart String
- 예를 들어서 경험치 아이템의 경험치 지급량을 바꾸고 싶다고 하자
- 지금은 `StringTable`에 해당 값을 직접 적어넣었기 때문에, 지급량이 바뀌면 텍스트도 바꿔야 함.
- `Unity Localization`은 `Smart String`을 지원한다. 
1. 테이블에서 `Smart`를 활성화하고 아래처럼 작성함
```
{value}레벨 달성 시 해금
Unlocked at level {value}
レベル{value}で解放
```
2. 코드로는 아래처럼 연결함
```cs
GetLocalizedSettings.StringDatabase.GetLocalizedString("UI_Strings", "ITEM_STAT_ATK",
    new { value = 10 });
```
> 테이블에서 0, 1로 달고 아래에서 변수 없이 값을 직접 순서대로 입력하는 것도 가능하지만, 이름이 있는 인자로 달아놓고 값을 연결해주는 게 좋다. 
> - 게다가 외국어인 경우는 순서가 바뀔 수도 있음.

3. **`Smart String`은 활성화된 요소에만 작동하므로 저 인자를 필요로 하지 않는 텍스트에도 멀쩡하게 작동함.**

4. 하지만 내 `ItemData`의 경우, 타입이 나뉘어 있고 하나는 수치값이 들어가지만 다른 건 수치값이 들어가지 않음. 그래서 코드로 분기를 명시해주는 게 더 좋다고 생각함. (더 확장할 계획은 없지만)
```cs
public string GetDesc()  
{
	if (_type == ItemType.Exp) 
	{
		return LocalizationSettings.StringDatabase.GetLocalizedString(
			LocalizationManager.TableItem, 
			DescKey, 
			arguments: new { value = _expAmount } // StringTable에서 Smart String을 쓰고 있으며 값은 value로 넣어놨음
		);
	}
	else
	{
		return LocalizationSettings.StringDatabase.GetLocalizedString(LocalizationManager.TableItem, DescKey);
	}
}
```

### 테스트
#### `ItemDatabase`에 없어서 제대로 초기화되지 않는 문제
```cs
private void LoadItemDatabase()
{
	#if UNITY_EDITOR 
	string[] guids = UnityEditor.AssetDatabase.FindAssets("t:ItemData",
		new[] { ITEM_GUID_PATH });
	foreach (string guid in guids)
	{
		string path = UnityEditor.AssetDatabase.GUIDToAssetPath(guid);
		ItemData itemData = UnityEditor.AssetDatabase.LoadAssetAtPath<ItemData>(path);
		if (itemData != null)
		{
			// itemData.name이 이슈였음
			itemDatabase[itemData.name] = itemData;
			Logger.Log($"{itemData.name} 아이템 DB에 추가됨");
		}
	}
	#endif
}
```
> 이전에 무심코 지나갔던 부분인데, `itemData.name`은 내가 설정한 이름이 아니다. `itemData.ID`로 변경했음.

- 추가로, `#if UNITY_EDITOR` 부분이 걸린다. 
	- 여러 방법을 알아봤으나, 아이템 갯수도 적으니 직접 할당하는 걸로 진행함. `itemDataList`를 인스펙터 단위에서 초기화 및 할당하고, `itemDatabase`를 런타임때 초기화하는 방식이다. 

#### 초기화 이슈 해결 후
```cs
return LocalizationSettings.StringDatabase.GetLocalizedString(
	LocalizationManager.TableItem, 
	DescKey, 
	arguments: new { value = _expAmount } // StringTable에서 Smart String을 쓰고 있으며 값은 value로 넣어놨음
);
```
> - 위 코드가 잘 동작하는지 테스트
> - 반드시 Smart String이 켜져 있어야 함. 안 그러면 오류가 발생한다.

- 이외에도 `ITEM_PROMOTION_CHIP`으로 설정되어 있지 않아서 발생하는 문제 해결
![[Pasted image 20260522235426.png]]
> 이런 느낌으로 들어감.


### 테스트 후, 위 요소인 ItemInfoPopup도 작업
- 사실 얘가 훨씬 중요한 건데 엌ㅋㅋ 
- `ItemUIElement`는 크게 상관없다. 실제로 상태에 텍스트들이 영향을 받는 건 이 부분임

- 코드로 관리되는 부분은 아래처럼 수정
```cs
protected override void Awake()
{
	base.Awake();
	SetUpPanel();

	LocalizationSettings.SelectedLocaleChanged += OnLocaleChanged;
}

void OnLocaleChanged(Locale locale) => Refresh();

void Refresh()
{
	itemNameText.text = currentItem.GetName();
	itemDescriptionText.text = currentItem.GetDesc();
}

protected override void OnDestroy()
{
	LocalizationSettings.SelectedLocaleChanged -= OnLocaleChanged;
}
```

- 코드와 관계없이 언어에 의해서만 변하는 부분(`보유`)은 `Localize String Event`로 반영.

### Item에 관한 StringTable까지 작성하고 테스트
> 영어
![[Pasted image 20260523000724.png]]

> 일어
![[Pasted image 20260523000749.png]]

> 한국어
![[Pasted image 20260523000802.png]]

- 일단 오늘은 여기까지!!!!!!!!!!!! 오늘도 변함없이 생각보다 오래 걸렸다.