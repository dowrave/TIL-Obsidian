

>[!question]
>메서드 호출 시 `null`에 따른 분기를 어디서 주는 게 좋을까?

```cs
// 예시 1. 외부에서 분기를 주기
MyType value = MyClass.MyMethod();
if (value != null)
{
	MyClass.MyMethod2(value);
}

// 예시 2. 메서드 내부에서 분기를 주기
public class MyClass
{
	public void MyMethod2(MyType input)
	{
		if (input != null) 
		{
			// ...
		}
	}
}
```

## 원칙
>[!note]
>`null`의 **의미를 해석할 수 있는 계층**에 둔다.

- `null`에는 크게 2가지 의미가 있다.
1) 의미 있는 도메인 상태로서의 `null` - "빈 세이브 슬롯" 등, `null` 자체가 하나의 유효한 상태
2) 잘못된 입력, 에러로서의 `null` 

**(1)번의 경우는 메서드 내부**(호출되는 쪽 : `콜리Callee`)에 표시하는 게 맞고, **(2)번의 경우는 메서드 외부**에 표기하는 게 맞음(즉 `null`이 에러로서 떴다면 아예 메서드를 호출하지 않는 방향)