>[!note]
>- `Toggle Group` 컴포넌트에 묶인 여러 `Toggle` 컴포넌트 중 하나를 선택할 때 사용
>- `Toggle`은 `bool isOn`으로 자신이 선택된 상태임을 관리하는 개별 요소
>- `Toggle Group`은 여러 `Toggle`을 관리하는 요소.
>	- `Allow Switch Off` : 여러 토글 중 하나를 선택하는 개념이 아니라, 각 토글의 `On/Off`을 따로 관리할 수 있게 만드는 개념.

## 예시

```
SaveSlotPanel
- Layout
-- SaveSlot
-- SaveSlot(1)
-- SaveSlot(2)
```
- **여러 개 중 어느 하나만을 선택하는 과정을 작성하고 싶다면, `Toggle/ToggleGroup`이 있다.** 

### Toggle
- On/Off 상태를 가진 `Button`. 
	- `bool isOn` 상태를 들고 있음
	- 클릭시 `isOn`이 반전, `OnValueChanged` 이벤트가 발생한다. 
		- `UnityEvent<bool>`임.
	- `Group` 필드에 `ToggleGroup`을 지정하면 같은 그룹 내 다른 `Toggle`이 켜질 때 자신은 자동으로 꺼진다. 

### ToggleGroup
- 여러 `Toggle`이 같은 `ToggleGroup`을 참조하면 그중 하나만 켜지도록 강제준다.
- `Allow Switch Off` 
	- 이미 선택된 토글을 다시 클릭했을 때 `Off`로 바뀌는지 여부. 


### 사용법
```
SaveSlotPanel
  Layout            <- 여기에 ToggleGroup 컴포넌트 추가
    SaveSlot        <- Toggle 컴포넌트 추가, Group = Layout의 ToggleGroup
    SaveSlot (1)    
    SaveSlot (2)    
```
1. 그룹에 `ToggleGroup` 컴포넌트 추가
2. 각 `Toggle` 요소에 `Toggle` 컴포넌트 추가, `Group`을 상단에 지정한 `ToggleGroup`으로 설정

- 현재 선택된 요소를 가져오는 방법 2가지
1. `ToggleGroup.ActiveToggles()`로 현재 켜진 토글을 가져온 다음, `GetComponent`로 컴포넌트 잡기
2. `SaveSlot`들을 별도로 관리하고 있다면 순회해서 현재 켜진 토글값 하나 가져오기
