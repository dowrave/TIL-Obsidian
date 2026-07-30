
### 상황 설정

```cs
public abstract class UnitEntity 
{
	protected void method1 
	{
		method2();
	}

	private void method2 
	{
		Debug.Log("헤헤");
	}
}

public class Entity2: UnitEntity 
{
	public void method3 
	{
		method1();
	}
}
```

자식 엔티티에서 부모 클래스의 메서드를 실행시킨 위의 상황에서 `method2`는 실행되는가? : ㅇㅇ

## 접근자Access Modifier
- 그 요소를 **직접** 호출, 접근할 수 있는지에 대한 것
	- `직접` : 그 멤버의 이름을 명시적으로 사용하여 호출하거나, 값을 읽고 쓰는 것 

- 더 넓은 접근 범위를 가진 멤버로 더 제한된 접근 범위의 멤버에 접근하는 건 가능하다.
	- 이를 `간접 접근`이라고 할 수 있으며, 제한을 우회하지 않는다. 

- 캡슐화의 핵심 도구다.
	- 내부 구현 세부사항은 숨기고`private`, 필요한 인터페이스만 외부에 노출`public`해서 코드의 안정성, 유지보수성을 높일 수 있다. 