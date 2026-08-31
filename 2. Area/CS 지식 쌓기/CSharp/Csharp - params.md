#Csharp 

## 설명

- 메서드의 매개변수 개수를 호출하는 쪽에서 가변적으로 넘길 수 있게 해주는 키워드.
```cs
public static void AddClickSound(params Button[] buttons)
{
	foreach (Button button in buttons)
	{
		AddClickSound(button);
	}
}
```

위 메서드는 만약 `params`가 없다면 아래의 배열을 만들어서 넘겨야 함.
```cs
AddClickSound(new Button[] { btnA, btnB, btnC })
```

하지만 `params`이 있기 때문에 아래처럼 처리할 수 있다.
```cs
AddClickSound(btnA, btnB, btnC);
```

- 묶는 건 컴파일러가 알아서 해줌.

### 제약
- 매개변수 목록에서 반드시 마지막에 와야 함
- 한 메서드에 하나만 쓸 수 있음
- 인자를 아예 안 넘기면 빈 배열로 처리됨