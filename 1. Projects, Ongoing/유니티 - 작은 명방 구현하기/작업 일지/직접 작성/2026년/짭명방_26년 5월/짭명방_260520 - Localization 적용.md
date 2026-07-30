
## AI 요약

**Unity Localization (StringTable) 적용 작업**을 진행했습니다.

#### 구조 설계

- StringTable을 용도별로 분리 (`UI_Strings`, `Stage_Strings` 등)
- `LocalizationManager`는 테이블 관리보단 편의 메서드 제공 역할로 정리
- 적용 방식을 두 가지로 확립
    - 상태에 따라 달라지는 텍스트 → 코드로 `GetLocalizedString(table, key)` 호출
    - 고정 텍스트 → 인스펙터에서 `Localize String Event` 사용

#### 주요 작업

- `StageData`의 `StageName`, `StageDesc` 필드를 StringTable 키 기반으로 교체, 필드도 `[SerializeField] private + 게터` 구조로 리팩토링
- `KeybindingSection`, `OperatorName` 등 기존 `GetText()` 방식 Localization으로 교체
- `SquadEditPanel`, `OperatorInventoryPanel`, `OperatorDetailPanel`, `OperatorLevelUpPanel` 작업 완료

#### 이슈 및 해결

- 코드로 텍스트를 할당하는 경우 **언어 전환 시 즉시 반영 안 됨** → `SelectedLocaleChanged` 이벤트 구독으로 해결
- `OperatorPromotionPanel`에서 하드코딩된 한국어 텍스트 발견 → StringTable 키로 분리 및 이벤트 구독 적용
- 일본어 폰트 Atlas/SDF 연결 이슈 → SDF 대신 새로 생성한 Atlas 파일 연결로 해결

#### 남은 작업

- `OperatorPromotionPanel` 영어 텍스트 레이아웃 문제 (공격 범위 영역 침범)
- `MainMenuScene` : `ItemInventoryPanel`, 각종 팝업, 튜토리얼 문구
- `StageScene` 전체 (아직 미착수)
- `Operator` 스킬 정보

---


>[!note]
>- TMP들에 폰트 애셋 세팅은 다 해놓은 상태
>	- 각각의 TMP 요소에 넣을 텍스트 할당하기(일단 한국어로만)
>	- 나머지 언어들은 AI 돌리고 부족한 실력으로 검수 조금 하는 정도?

- 근데 적용하려고 하니까 시작부터 애매한 게 하나 있다
	- 기존에는 `StageData`에서 텍스트들을 긁어서 옮기는 방식이었음
	- 이걸 `StringTable` 기반으로 옮긴다면, `StageData`에선 `Key`값을 갖게 하면 되나?

---

## StringTable 관리 방법
- 용도에 따라 만들어두는 게 좋음
```
📁 Tables/
  UI_Strings       ← 버튼, 레이블 등 UI 텍스트
  Stage_Strings    ← 스테이지 이름, 설명
  Item_Strings     ← 아이템 이름, 설명
  Dialog_Strings   ← 대화, 스토리 텍스트
```

- 이렇게 만든 **테이블은 패키지가 관리**하므로, 싱글턴 매니저에서 별도로 들고 있을 필요는 없음

```cs
// 테이블 이름 + 키만 알면 어디서든 바로 호출 가능
LocalizationSettings.StringDatabase.GetLocalizedString("Stage_Strings", "STAGE_1_0_NAME");
```
> - 현재 설정된 언어는 `LocalizationSettings.SelectedLocale`에 있음

- `LocalizationManager`에서는 테이블 관리보다는, 편의성을 제공하는 역할로 분리하면 된다.
```cs
// 자주 쓰는 테이블 이름 상수화
public const string TableUI     = "UI_Strings";
public const string TableStage  = "Stage_Strings";
public const string TableItem   = "Item_Strings";

// 편의 메서드
public string GetUI(string key)    => LocalizationSettings.StringDatabase.GetLocalizedString(TableUI, key);
public string GetStage(string key) => LocalizationSettings.StringDatabase.GetLocalizedString(TableStage, key);
```


---
## 적용
- `StageData`에 있는 `StageName, StageDesc`을 `Localization Table`인 `StringTable`로 옮김
![[Pasted image 20260520142609.png]]

- 그리고 해당 필드의 이름을 `stageNameLocKey, stageDescLocKey`로 변경

> [!note]
> - 이 과정에서 `StageData`의 모든 필드가 `public`이었던 관계로, `[SerializeField] private + 게터` 프로퍼티 구조로 바꿨음
> - 이런 노가다 부분은 AI 쓰는 게 확실히 편한데, 말을 잘 전달해야겠다. 게터 프로퍼티 얘기를 빼놓으니까 아예 안해주네..

- 다른 경우에도 적용하면 좋을 것 같아서 `StageData`에 관한 예시를 적어둠
```cs
    [Header("Stage Description")]
    [Tooltip("{a}-{b}이라는 규칙으로 사용 중")]
    [SerializeField] private string stageId = string.Empty;

    // 규칙 기반
    private string stageInfoLocTable = "StageInfo"; // StringTable 이름
    
    public string StageNameLocKey => $"STAGE_{stageId}_NAME";
    public string StageDescLocKey => $"STAGE_{stageId}_DESC";
    
    // 기존 `TMP.text`에 메서드로 집어넣으면 됨
    public string GetName() => LocalizationSettings.StringDatabase.GetLocalizedString(stageInfoLocTable, stageNameLocKey);
    public string GetDesc() => LocalizationSettings.StringDatabase.GetLocalizedString(stageInfoLocTable, stageDescLocKey);

```

> `StageSelectPanel`에서 "현재 선택된 스테이지의 버튼"에 따라 우측 패널에 들어가는 글자가 달라져야 하는 상황이라면 이전에 썼던 `Localize String Event`만으로는 처리할 수 없는 상황임
> 	- 언어에 따라 텍스트가 달라지는 상황만 처리하고 싶다면 저걸로만 하면 충분함. 예를 들어서 아이템의 보상 패널 위에 "Reward/보상"으로 구분하고 싶은 상황. 이거는 언어에 따라서만 달라지는 상황이니까.

## 전체적인 흐름 잡기 & 작업

>[!note]
>- **크게 2가지 방법으로 갈 듯**
>	- **상태에 따라 달라지는 경우 `GetLocalizedString(table, key)`**
>	- **그렇지 않은 경우 인스펙터에서 `Localize String Event`로 사용**
- 참고 : `GetLocalizedString`을 쓰는 경우 게임 뷰에서 언어를 바꿨을 때 텍스트가 즉시 바뀌는 개념은 아님! 이벤트를 구독해야 함
```cs
// 대충 느낌만 적어놓음 : Localization 텍스트를 사용하는 UI 요소들에 넣어야 함
LocalizationSettings.SelectedLocaleChanged += OnLocaleChanged;

void OnLocaleChanged(Locale locale) => Refresh();

void Refresh()
{
    stageNameText.text = _currentStageData.GetName();
}
```

일단 **StageInfo 테이블 적용 완료**

---
위 과정에서 `LocalizationManager`의 기존 기능을 `Legacy`로 분리하고 새로운 `LocalizationManager`로 넣었는데, `GetText()`를 이용하던 기능들에서 오류가 발생하기 시작했다. 주로 **`OperatorData, DeployableData` 관련**임


### KeybindingSection
```cs
[KeyActionLabel("keybinding.returntolobby")]
ReturnToLobby,
```
> 어트리뷰트로 들어간 `keybinding.returntolobby` 같은 부분 때문에 그렇다. 기존엔 `localizationManager`에서 키값을 찾는 식으로 수행하기 위해 저런 식으로 명명했는데, 지금은 기능에 따른 테이블이 분리되고 있고 대문자로 명명하고 있어서 저 값들을 바꿔줘야 함


- `Operator_Name`, `Keybinding` 부분을 `Localization`으로 바꾸는 것까지 진행했음
- 프로젝트 여기저기 계속 보면서 일반 텍스트를 `Localization` 기반으로 바꿔줌 
	- 여기엔 정리한 내역들 쓰는 식으로 작업. 여기 쓴 텍스트에 비해 한 일이 많을 거다..

#### KeyBindingButtonContainer


### 일단 여기서
- 위에서 코드 기반 `GetLocalizedString`으로 짰을 경우에, 언어 전환 시 즉시 반영되지 않는 문제가 있었음
```cs
// 대충 느낌만 적어놓음 : Localization 텍스트를 사용하는 UI 요소들에 넣어야 함
LocalizationSettings.SelectedLocaleChanged += OnLocaleChanged;

void OnLocaleChanged(Locale locale) => Refresh();

void Refresh()
{
    stageNameText.text = _currentStageData.GetName();
}
```
> 이런 이벤트 구독과 해제 코드가 필요하다~
> 그래서 지금까지 작업한 요소들에 대해서 다시 적용하는 과정을 진행함
- `StageSelectPanel`
- `KeybindingSection`(`OptionPopup`)
- `OperatorSlot`

### 계속 진행
- **참고 : `OperatorSkill`, `StageData`, `ItemData` 등등은 나중에 진행할 예정**
- `SquadEditPanel` 완료
- `OperatorInventoryPanel` 완료
- `OperatorDetailPanel` 완료
- `OperatorLevelUpPanel` 완료
- `OperatorPromotionPanel` 작업 중

#### PromotionPanel 작업 중 이슈
- 태그가 들어가면 제대로 안 들어가는 것 같은데?
![[Pasted image 20260520231829.png]]
> - 언어를 `JP`로 설정한 상태에서 외부에서 진입했을 때의 화면.
> - "현재 정예화", "스킬 사용 가능", "정예화 조건을 달성하지 못했습니다" 등은 모두 일본어로 나타나야 하는데 나타나지 않고 있음

저 화면 자체에서 다른 언어로 바꿨다가 다시 `JP`로 바꾸면 잘 반영됨
![[Pasted image 20260520231929.png]]
> 여전히 한국어인 부분은 반영을 안한 상태, 0이 작은 건 테스트용

-  태그 문제 아님 -> 하단의 "필요 아이템"으로 테스트해봤고, 태그 반영 잘 됨

- **스크립트 단위에서의 문제로 보인다** : 스크립트를 보니까 실제로 이런 게 있음
```cs
OperatorElitePhase currentPhase = op.CurrentPhase;
OperatorElitePhase newPhase = op.CurrentPhase + 1; // enum이니까
currentPromotionText.text = $"<size=200>{(int)currentPhase}</size>\n\n현재 정예화";
newPromotionText.text = $"<size=200>{(int)newPhase}</size>\n\n목표 정예화";
```
> 텍스트를 할당하는 부분은 다 지워주겠음

- 그런데 "정예화를 하지 못할 조건"으로 아래처럼 코드가 있었음
```CS
private void SetCannotConditionText()
{
	string baseText = "정예화 조건을 달성하지 못했습니다.\r\n필요 조건 : <color=#FFCCCC>";

	// 레벨이 부족한 경우
	if (!op.CanPromote)
	{
		string lowLevelText = "0정예화 50레벨</color>";
		cannotConditionText.text = baseText + lowLevelText;
	}
	// 정예화 아이템이 없는 경우
	else if (opData != null && !GameManagement.Instance!.PlayerDataManager.HasPromotionItems(opData))
	{
		// 요구조건이 다 동일할 예정
		string lowLevelText = "정예화 아이템 1개</color>";
		cannotConditionText.text = baseText + lowLevelText;
	}
}
```
> - 각각의 경우를 `StringTable`에서 키로 분리하고 텍스트를 가져오면 된다. 
> - 코드에서 텍스트를 채우기 때문에 언어 변경 이벤트를 구독해줘야 함(기존엔 Localization 자체로 해결했으나 코드 단위에서 변경하겠다면 이벤트 구독이 필요함)

- AI가 던져준 기본 코드는 이건데
```cs
void Start()
{
    LocalizationSettings.SelectedLocaleChanged += _ => SetCannotConditionText();
}

private void SetCannotConditionText()
{
	string baseText = LocalizationSettings.StringDatabase.GetLocalizedString("UI", "PROMOTION_CANNOT_BASE");

	// 레벨이 부족한 경우
	if (!op.CanPromote)
	{
		string condText = LocalizationSettings.StringDatabase
			.GetLocalizedString("UI", "PROMOTION_CANNOT_LEVEL");
		cannotConditionText.text = $"{baseText}<color=#FFCCCC>{condText}</color>";
	}
	// 정예화 아이템이 없는 경우
	else if (opData != null && !GameManagement.Instance!.PlayerDataManager.HasPromotionItems(opData))
	{
		string condText = LocalizationSettings.StringDatabase
			.GetLocalizedString("UI", "PROMOTE_CANNOT_ITEM");
		cannotConditionText.text = $"{baseText}<color=#FFCCCC>{condText}</color>";
	}
}

void OnDestroy()
{
    LocalizationSettings.SelectedLocaleChanged -= _ => SetCannotConditionText();
}
```

- 이거 말고도 고칠 게 더 있어서 아래의 구조로 수정
```cs
private void Start()
{
	LocalizationSettings.SelectedLocaleChanged += OnLocaleChanged;
}

private void OnLocaleChanged(Locale locale) => Refresh();
private void Refresh()
{
	OperatorSkill? unlockedSkill = opData.Elite1Unlocks.unlockedSkill;
	if (unlockedSkill != null)
	{
		string condText = LocalizationSettings.StringDatabase.GetLocalizedString("UI", "PROMOTION_ADDITIONAL_SKILL");
		skillNameText.text = $"{condText}<color=#179bff>{unlockedSkill.SkillName}</color>";
	}

	SetCannotConditionText();
}
```

> [!note]
> - 일본어 폰트를 더 큰 `Atlas`로 만들고 `Fallback`으로는 `SDF` 파일을 연결해도 된다고 `Claude`가 알려줬는데, 실사용 시에 문제가 있었다. 가타카나가 ㅁ으로 잘리는 현상이 발생했음.
> - `SDF` 파일 대신 새로 생성한 `Atlas` 파일을 연결해주면 해결된다. `SDF`와 `Atlas`가 바로 연결되는 게 아닌건가?

- **코드로 처리되는 부분들은 컴포넌트 `Localize String Event`를 제거해줬다.**

#### 일단 여기까지
- 프로젝트의 텍스트 구조를 다 갈아엎는 일이라 오래 걸린다.
- `OperatorPromotionPanel`에서 추가로 할 일
	- 공격 범위 표시 시 한국어/일본어는 괜찮은데 영어는 길어서 공격범위 영역을 침범함
	- 아래 둘 중 하나의 방식으로 해결할 수 있을 듯
		- 공격범위 영역을 텍스트 영역의 아래로 빼든지
		- 텍스트 영역을 인식해서 공격범위 영역이 움직이든지
- 남은 거
	- `MainMenuScene` 
		- `ItemInventoryPanel` 현재의 이슈
		- 팝업들 `ItemInfoPopup`, `OpRoleInfoPopup`
		- 튜토리얼 문구
	- `StageScene` - 1개도 작업 안 함 ㄱㄱ
	- `Operator`의 스킬 정보들
