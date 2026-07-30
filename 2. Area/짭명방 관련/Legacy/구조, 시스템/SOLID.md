
> 예시 코드
```cs
public abstract class Parent 
{
	protected float var1;

	public virtual void Initialize(float inputVar1) 
	{
		var1 = inputVar1;
	} 
}

public class Child: Parent 
{
	private float var2 

	public override void Initialize(float inputVar1, float inputVar2)
	{
		base.Initialize(inputVar1);
		var2 = inputVar2;
	}
}
```
> 이 코드를 보면 부모 클래스에서 `Initialize`를 구현하지 않고, 자식 클래스에서만 `Initialize`를 하는 게 코드의 글자 수 등에서 더 효율적으로 보이지만, 이 방식이 더 객체 지향 설계에 더 맞다고 한다. 

이게 뭔 소리인가 보고자 SOLID 를 알아본다.

## S : Single Responsibility Principle(단일 책임 원칙)

> 클래스는 하나의 책임만 가져야 하며, 이를 변경할 이유도 하나여야 한다.

- 부모 클래스는 공통 필드의 초기화 작업을 담당하고 자식 클래스는 고유 필드의 초기화만 담당한다.
- 책임이 분리되어 있기 때문에 부모 클래스는 공통적인 초기화 로직에 대한 책임을, 자식 클래스는 자신만의 초기화 로직에만 집중할 수 있다.
- 각 클래스가 자신만의 책임을 갖기 때문에 SRP를 충족한다.

## O : Open / Closed Priniciple (개방 - 폐쇄 원칙)

> 클래스는 확장에 열려있어야 하고, 수정에 닫혀 있어야 한다

- 자식 클래스는 부모 클래스에서 기본 초기화 로직이 구현되어 있기 때문에, 오버라이드만으로 확장이 가능하다.
- **부모 클래스의 코드를 변경하지 않고 자식 클래스에서 기능을 추가**할 수 있기 때문에 이를 만족한다.

## L : Liskov Substitution Principle (리스코프 치환 법칙)

> 자식 클래스는 부모 클래스를 대체할 수 있어야 한다

- 자식 클래스의 오버라이드한 Initialize만으로 부모 클래스의 기능을 사용하면서 부모 클래스를 대체할 수 있기 때문에 이를 만족한다.

## I : Interface Segregation Principle (인터페이스 분리 원칙)

> 클라이언트는 자신이 사용하지 않는 메서드에 의존해서는 안 된다

- 부모 클래스는 자식 클래스가 필요로 하는 메서드만 제공하므로, 자식 클래스는 필요한 메서드만 오버라이드하거나 구현하게 된다.
- 각 클래스는 필요 없는 기능에 의존하지 않게 된다.
## D : Dependency Inversion Principle (의존 역전 원칙) 

> 고수준 모듈은 저수준 모듈에 의존해서는 안되며, 두 모듈 모두 추상화에 의존해야 한다.

- 부모 클래스는 추상화된 `Initialize`를, 자식 클래스는 구체화된 `Initialize`를 구현하거나 확장한다. 

---

이러한 SOLID 원칙은 부모 - 자식 간의 관계에도 적용된다. 
- 부모 클래스는 공통적인 초기화, 필드 관리
- 자식 클래스는 고유한 필드를 처리
