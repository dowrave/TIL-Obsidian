#Unity 

> 이벤트를 발생시킬 때 파라미터 관련 설정이 헷갈리기 시작했음

1. 이벤트 설정
```cs
private Action<T> listeners = new Action<T>();
```

2. 이벤트 발생
```cs
listeners.Invoke(parameter);
```
> 여기서 할당될 파라미터가 선정된다. 


3. 이벤트에 리스너 추가
```cs
listeners.AddListener(() => method());
```


--- 
내 상황
- `OperatorSlotButton`
```cs
    // OperatorSlotButton 타입의 파라미터를 받는 이벤트 정의
    public UnityEvent<OperatorSlotButton> OnSlotClicked = new UnityEvent<OperatorSlotButton>();

    private void Awake()
    {
        if (button == null) button = GetComponent<Button>();

        // Button 클릭 시 OnSlotClicked 이벤트 발생, 현재 OperatorSlotButton(this)을 파라미터로 전달함
        button.onClick.AddListener(() => OnSlotClicked.Invoke(this));
    }
```

- `SquadEditPanel`
```cs
private void InitializePanel()
{
   // 슬롯별 타입 설정
   for (int i = 0; i < operatorSlots.Count; i++)
	{
		OperatorSlotButton slot = operatorSlots[i];
		
		// 1. 활성화
		if (i < ACTIVE_OPERATOR_SLOTS)
		{
			slot.Initialize(true);
			
			slot.OnSlotClicked.AddListener(HandleSlotClicked);
			// 아래는 기능은 동일하지만 필요 없는 Wrapper 함수만 하나 더 생김
			// slot.OnSlotClicked.AddListener((slot) => HandleSlotClicked(slot));
		}
		else
		{
			slot.Initialize(false);
		}
	}
}
```