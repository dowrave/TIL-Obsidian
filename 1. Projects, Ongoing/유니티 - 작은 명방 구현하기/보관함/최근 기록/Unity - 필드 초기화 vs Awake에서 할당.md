#Unity 

```cs
public class script 
{
	public string pathIndicatorTag1 = "pathIndicator1";
	public string pathIndicatorTag2;
	
	private void Awake() 
	{
		pathIndicatorTag2 = "pathIndicator2"
	}
}
```

위 2가지 방법에는 어떤 차이가 있을까?

## 1. 필드 초기화
- **실행 시점** :  객체가 메모리에 생성되는 순간(C# 생성자 단계)에 실행된다. 
	- `Awake()`보다 먼저 실행된다.
- 코드가 간결하고, 변수의 정의와 값이 함께 있어 가독성이 좋다.

## 2. Awake() 할당
- 실행 시점 : 유니티 엔진이 해당 스크립트를 활성화하고 초기화하는 단계에서 실행된다.
- 특징 : 다른 컴포넌트를 참조할 필요가 있거나, 런타임 로직에 따라 값을 결정할 때 쓴다.
- 단순한 문자열 리터럴 할당에 Awake를 쓰는 건 **불필요한 오버헤드이다.**

## 권장
- 변하지 않는 값을 설정할 경우, 아예 `const`까지 붙여서 필드 초기화를 하는 걸 추천.
	- `const`를 붙이면 접근자도 `public`으로 노출시켜도 된다. 변하지 않으니까.

```cs
public const string PathIndicatorTag = "PathIndicator";
```
> - C#에서 `const`로 선언된 변수는 자동으로 `static` 속성을 갖게 된다. 즉, 인스턴스가 아닌 **클래스 자체에 소속되는 변수가 된다.**

따라서, 해당 필드에 접근할 때는 싱글턴 인스턴스임에도 `Instance`를 생략할 수 있음!
```cs
ObjectPoolManager.PathIndicatorTag
```
