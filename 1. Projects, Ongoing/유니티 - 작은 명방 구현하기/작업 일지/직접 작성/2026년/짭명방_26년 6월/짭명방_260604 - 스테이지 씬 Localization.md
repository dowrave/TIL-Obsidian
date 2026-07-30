>[!done]
>- 어제 못한 거 고치기
>	- 인게임에서 언어 변경 시 발생하는 오류 수정
>	- 언어 변경 이벤트 구독받지 않는 요소들 구독시키기
>- 스테이지 씬 `Localization` 기반으로 고치기
>	- `InStageInfoPanel`
>		- 오퍼레이터 선택 / 배치 요소 선택에 따라 데이터를 가져오는 곳이 다르고 해서 생각이 복잡해졌음. 처리 완료.
>- 기타 이슈 수정
>	- `TestManager`의 스테이지 클리어 로직이 제대로 동작하지 않는 현상 수정
>		- `StageId`의 포맷이 바뀌면서 하드코딩했던 요소들에서 이슈가 발생했다. 첫 스테이지만 하드코딩으로 남겼고, 나머지는 `StageData`에서 이전 스테이지 값을 저장하는 식으로 수정함
>	- `StatPanel` 토글 애니메이션이 인게임 타임스케일에 영향받는 현상 수정
>	- 언어 선택 드롭다운의 각 버튼이 소리나게끔 구현 

## 어제 못한 거 고치기
>[!note]
>- 6월 3일 못한 것
>- 언어 변경 시 뭐가 반영되지 않는 것으로 보임(테이블? 뭔지 모르겠다. 오류 내용도 체크할 것)
>- 언어 변경 이벤트 구독받지 않는 요소들 구독시키기
### 언어 변경 시 발생하는 오류 수정

- 요약
	- **원인** : `ConfirmationPopup`에서 발생했던 이슈로, `currentTable, currentKey`라는 `string`값이 할당되기 전에 `Locale` 변경 이벤트를 구독했다. 그래서 `null`값들이 들어갔기 때문에 발생함.
	- **해결** : 이벤트 구독 타이밍을 기존의 `Awake`에서 `OnEnable`로 맞춰주고, 메서드에도 인풋 파라미터가 `null`이면 동작하지 않게 수정함
#### 오류 내용

- 게임 뷰에서 `Localization`의 기능을 이용해서 수정할 때와 달리, 인게임에서 

```cs
languageDropdown.onValueChanged.AddListener(GameManagement.Instance.LocalizationManager.SetLocale);
```

으로 동작하는 코드는  아래 에러를 발생시킴

```
ArgumentException: Empty Table Reference. Must contain a Guid or Table Collection Name
```

- 스택이 길어서 AI에게 던져봤음
```
TMP_Dropdown 값 변경
  → LocalizationManager.SetLocale()
    → LocalizationSettings.SelectedLocale 변경
      → OnLocaleChanged() 콜백 호출  ← 여기서 문제 발생
        → ConfirmationPopup.Refresh()
          → ConfirmationPopup.SetTextContent(table, key)
            → table이 비어있어서 예외 발생
```

- 코드를 따라가보면 아래의 구성.
```cs
private void Awake()
{
	LocalizationSettings.SelectedLocaleChanged += OnLocaleChanged;
}

private void OnLocaleChanged(Locale locale) => Refresh();
private void Refresh() => SetTextContent(currentTable, currentKey);

private void SetTextContent(string table, string key)
{
	string contents = LocalizationSettings.StringDatabase.GetLocalizedString(table, key);
	textContent.text = contents;
}
```
- 즉 `currentTable, currentKey`가 할당되지 않은 상태에서 `Refresh`가 들어가면서 발생한 이슈.

#### 해결
```cs
private void SetTextContent(string table, string key)
{
	// 조건 추가
	if (string.IsNullOrEmpty(table) || string.IsNullOrEmpty(key)) return;
	
	string contents = LocalizationSettings.StringDatabase.GetLocalizedString(table, key);
	textContent.text = contents;
}
```
- `SetTextContent`가 동작하는 조건만 추가해줌

- 추가로, 로케일 변경 이벤트를 구독하는 시점을 활성화된 시점으로 바꿔줌
	- `Initialize`에서 `currentTable, currentKey`가 할당되기 때문임
	- 이 구현은 `OnEnable`에 해도 되고 `Initialize`에 해도 된다. 활성화는 `Initialize` 내부에서 진행하기 때문임.

- 이벤트 구독 / 해제 쌍에 대해서.

| 구독 위치      | 해제 위치       | 적합한 상황                  |
| ---------- | ----------- | ----------------------- |
| `OnEnable` | `OnDisable` | 활성/비활성을 반복하는 오브젝트       |
| `Start`    | `OnDestroy` | 생애주기 전체에 걸쳐 한 번만 필요한 경우 |
| `Awake`    | `OnDestroy` | 아주 이른 시점부터 필요한 경우       |

> 팝업의 경우는 `OnEnable / OnDisable` 쌍이 적합하다고 함. 

### 메인메뉴 씬 : 언어 변경 이벤트 구독시키기
- 띄워놓고 써먹기 위해 여기에도 적어둠
```cs
// 복습
// Awake나 OnEnable에
LocalizationSettings.SelectedLocaleChanged += OnLocaleChanged;

private void OnLocaleChanged(Locale locale)
{
	// 고칠 요소들
}

// OnDisable이나 OnDestroy에
LocalizationSettings.SelectedLocaleChanged -= OnLocaleChanged;
```
> `Refresh()`도 필요없어 보이니까 다 고쳐놓음. 굳이 이중으로 해놓을 이유가 없다. 
> 	Refresh()라는 메서드 이름이 너무 흔하게 쓰여서 뚜렷한 목적성이 없어 보이기도 함.

- 어제는 구독되지 않은 부분이 있다고 생각했는데, **인게임에서의 기능으로만 테스트했을 때 눈에 띄는 부분은 보이지 않음**
	- 에디터에서는 인게임에서 언어 변경이 불가능한 상황에서도 바꿀 수 있지만, 기준은 인게임에서 봤을 때 자연스러우면 됐다고 생각함. 그래서 모든 요소에 일일이 구독을 달아두지는 않겠음.
	- 물론 나중에 뜻밖의 상황이 생길 수도 있다.

## 스테이지 씬 Localization
- 드디어 스테이지 씬도 고친다. 자꾸 딴 곳으로 새서;;

```cs
// SmartString을 쓰는 경우 GetLocalizedString 예시
remainDeploymentCountText!.text = LocalizationSettings.StringDatabase.GetLocalizedString(
	LocalizationManager.UI.TableName,
	LocalizationManager.UI.RemainDeploymentCount,
	arguments: new { value = remainDeploymentCount }
);
```

### InstageInfoPanel
```
체력 : 
공격력 : 
방어력 : 
```
이걸 `StringTable`로 만든다고 치면

```
체력(TMP) :(TMP)
```
으로 구성할까? 아니면

```
체력 :(TMP)
```
로 구성할까? 가 궁금해졌음.

- 이번엔 제미나이에게 물어봤다. `Smart String`을 이용해서 한꺼번에 처리하는 걸 추천했음.
```
체력 : {value}(TMP)
```
> - `체력 : `에 들어가는 글자에 따라 서식이 어느 정도 달라질 여지가 있음
> - `<pos = 50%>` 리치 텍스트 태그가 있다고 해서 테스트해봄

- 생각보다 복잡한 구조라서 시간이 좀 끌리는 중

#### 구현 및 테스트

- 수정된 구조
```cs
// 조건 필드 추가
private bool IsEditingDeployable => currentDeployableInfo != null && currentDeployableInfo.ownedOperator == null;
private bool IsEditingOperator => currentDeployableInfo.ownedOperator != null;

private void OnLocaleChanged(Locale locale)
{
	if (IsEditingDeployable)
	{
		nameText.text = currentDeployableInfo.deployableUnitData?.GetName() ?? string.Empty;
	}
	else if (IsEditingOperator)
	{
		nameText.text = currentDeployableInfo.operatorData?.GetName();

		UpdateStatInfo();

		// 하단 스킬에 들어가는 값들도 수정이 필요함
	}
}

private void UpdateStatInfo()
{
	int attackValue;
	int defenseValue;
	int magicResistanceValue;
	int blockCountValue;
	string attackString;
	string defenseString;
	string magicResistanceString;
	string blockCountString;

	// 기존 오퍼레이터가 있었다면 구독 해제
	if (currentOperator != null)
	{
		currentOperator.Health.OnHealthChanged -= UpdateHealthText;
		currentOperator.OnStatsChanged -= UpdateOperatorInfo;
	}

	if (currentDeployableUnitState != null && currentDeployableUnitState.IsDeployed)
	{
		// 배치된 경우 실시간 정보를 가져옴
		currentOperator = currentDeployableInfo.deployedOperator;
		if (currentOperator != null)
		{
			UpdateHealthText(currentOperator.Health.CurrentHealth, currentOperator.Health.MaxHealth);

			attackValue = Mathf.FloorToInt(currentOperator.AttackPower);
			defenseValue = Mathf.FloorToInt(currentOperator.Defense);
			magicResistanceValue = Mathf.FloorToInt(currentOperator.MagicResistance);
			blockCountValue = Mathf.FloorToInt(currentOperator.MaxBlockableEnemies);

			// 이벤트 구독
			currentOperator.Health.OnHealthChanged += UpdateHealthText;
			currentOperator.OnStatsChanged += UpdateOperatorInfo;
		}
		else
		{
			Logger.LogError("currentOperator가 null임");
			return;
		}
	}
	else
	{
		// Box에 있는 정보를 가져옴 
		
		OperatorStats ownedOperatorStats = currentDeployableInfo.ownedOperator!.CurrentStats;

		float initialHealth = ownedOperatorStats.Health;
		UpdateHealthText(initialHealth, initialHealth);

		attackValue = Mathf.FloorToInt(ownedOperatorStats.AttackPower);
		defenseValue = Mathf.FloorToInt(ownedOperatorStats.Defense);
		magicResistanceValue = Mathf.FloorToInt(ownedOperatorStats.MagicResistance);
		blockCountValue = Mathf.FloorToInt(ownedOperatorStats.MaxBlockableEnemies);
	}

	attackString = LocalizationSettings.StringDatabase.GetLocalizedString(
		LocalizationManager.UI.TableName,
		LocalizationManager.UI.StatAttackFormat,
		arguments: new { value = attackValue }
	);
	defenseString = LocalizationSettings.StringDatabase.GetLocalizedString(
		LocalizationManager.UI.TableName,
		LocalizationManager.UI.StatDefenseFormat,
		arguments: new { value = defenseValue }
	);
	magicResistanceString = LocalizationSettings.StringDatabase.GetLocalizedString(
		LocalizationManager.UI.TableName,
		LocalizationManager.UI.StatAttackFormat,
		arguments: new { value = magicResistanceValue }
	);
	blockCountString = LocalizationSettings.StringDatabase.GetLocalizedString(
		LocalizationManager.UI.TableName,
		LocalizationManager.UI.StatAttackFormat,
		arguments: new { value = blockCountValue }
	);
			
	attackText.text = attackString;
	defenseText.text = defenseString;
	magicResistanceText.text = magicResistanceString;
	blockCountText.text = blockCountString;
}
```
> 일단 이렇게 구현해보고 테스트

- 서식이 깨지는 현상이 있음. 레이아웃 만짐
	- 리치 텍스트 `pos` 제거함. 왜 만질 때마다 달라지지?
	- 제거하는 편이 훨씬 깔끔하다. 언어별로 글자수 정도만 맞춰줘도 충분함. 그렇지 않아도 크게 상관은 없고.

- 바리케이드까지 잘 적용되는지 테스트

- `Operator` 선택 시
![[Pasted image 20260604233501.png]]
> 언어가 한국어로 설정돼 있는 건 무시하자. 왜 저렇게 됐는지는 모르겠는데 잘 설정되고 있음.

- `Deployable` 선택 시
![[Pasted image 20260604233751.png]]

### 기타 이슈 수정
- `TestManager` : 스테이지 스킵 기능 제대로 동작하지 않음
	- 클리어 처리는 되는데, 화면에 반영되지 않는 듯
	- ...`StageId` 바꾸면서 발생한 이슈로 보임. 서식을 바꿨기 때문에 기타 바꿔야 하는 코드도 발생함.

- 예를 들면 이런 거. `stageId`를 설정하는 규칙이 바뀌었기 때문에, 이런 코드를 일일이 찾아서 수정해야 함. 아래는 현재 상태에 맞게 코드를 수정한 내용.
```cs
// 언락 상태 확인.
public bool IsStageUnlocked(string stageId)
{
	// 1번째 스테이지는 점검하지 않음
	if (stageId == "STAGE_1-0") return true;

	// STAGE_{A}-{B}의 형태로, {B}값만 추출해서 점검할 것임
	string[] parts = stageId.Split("-");
	int stage = int.Parse(parts[1]);

	// 이전 스테이지가 클리어됐을 때에만 언락
	string previousStageId = $"STAGE_1-{stage - 1}";
	return IsStageCleared(previousStageId);
}

```
> 근데 이런 식으로 코드를 두기보다는, `stageData` 자체에서 이전 스테이지 정보를 갖게 하는 게 더 나아보인다 .

- `stageData`에 이전 스테이지 정보 추가하고 코드도 다시 설정
```cs
// 언락 상태 확인
public bool IsStageUnlocked(StageData stageData)
{
	// 1-0은 항상 클리어
	if (stageData.StageId == "STAGE_1-0") return true;
	
	// 이전 스테이지가 없으면 경고문구
	if (stageData.PreviousStage == null)
	{
		Logger.LogWarning("이전 스테이지가 할당되지 않음");
		return false;
	}

	// 이전 스테이지 클리어 조건만 점검
	return IsStageCleared(stageData.PreviousStage);
}
```
> 여전히 STAGE_1-0은 하드코딩이긴 한데 이래놔도 크게 상관은 없지 않을까... 나중에 StageId의 형식을 또 고치면 모르겠지만, 아마 그럴 일은 없을 듯

- `StatPanel` : 인게임 타임스케일에 영향을 받는 이슈 있었음. `SetUpdate(true)`를 추가.

- `OthersSection` : 언어 드롭다운의 각 버튼에 클릭 사운드 추가

1. 드롭다운 버튼에 사운드 추가
```cs
private void OrganizeLanguageDropdown()
{
	DropdownClickDetector clickDetector = languageDropdown.GetComponent<DropdownClickDetector>();
	clickDetector.onClicked += () => 
	{
		GameManagement.Instance.Sound.PlayButtonClick();
		StartCoroutine(AdjustDropdownList());
	};
}
```

2. 드롭다운의 각 항목에 사운드 추가
```cs
for (int i = 0; i < content.childCount; i++)
{
	Transform child = content.GetChild(i);

	// 하이어라키 상에서 활성화되어 있는 요소만 리스트에 추가
	// ...

	// 각 항목에 클릭 사운드 추가
	Toggle toggle = child.GetComponent<Toggle>();
	if (toggle != null)
	{
		EventTrigger trigger = toggle.gameObject.AddComponent<EventTrigger>();
		EventTrigger.Entry entry = new EventTrigger.Entry
		{
			eventID = EventTriggerType.PointerClick
		};
		entry.callback.AddListener(_ => GameManagement.Instance.Sound.PlayButtonClick());
		trigger.triggers.Add(entry);
	}
}
```

