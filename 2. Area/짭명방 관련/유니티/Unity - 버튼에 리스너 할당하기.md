#Unity 

막힌 상황 : 어떤 버튼을 클릭했을 때 `현재 수정 중인 칸`을 설정 - 이 때 "현재 수정 중인 칸"을 어떻게 지정할 수 있는가?
- 최종적으로 현재 수정 중인 칸에 다른 패널에서 Operator를 설정해서 넣는 방식
```
MainMenuManager
- SquadEditPanel
	- OperatorSlot(Button 컴포넌트 포함) (클릭 시 OperatorSelectionPanel로 전환)
- OperatorSelectionPanel
```
---
1. `MainMenuManager`에 아래의 메서드 구현
```cs
    // 오퍼레이터를 선택하는 패널을 띄울 때 현재 설정 중인 슬롯 설정
    public void ShowOperatorSelectionPanel(OperatorSlotButton clickedSlot)
    {
        currentEditingSlot = clickedSlot;
        ShowPanel(MenuPanel.OperatorSelect);
    }
```
> 각 패널이 형제 관계이므로, 패널을 비활성화하고 활성화하는 과정에서 정보를 유지하고 있어야 함 -> 이를 `MainMenuManager`에서 수행

2. `SquadEditPanel`에서, `OperatorSlot`을 클릭했을 때 `MainMenuManager`에 있는 `메서드`를 실행시키도록 설정

3. `SquadEditPanel`을 초기화할 때 2번의 메서드를 모든 `OperatorSlot`의 이벤트 리스너에 등록
```cs

```




