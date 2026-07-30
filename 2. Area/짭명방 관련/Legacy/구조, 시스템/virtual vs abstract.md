- `virtual`
	- 부모 클래스에서 제공됨
	- 자식 클래스에서 재정의할 필요 없음
	- 필요시 자식 클래스에서 재정의 가능

- `abstract`
	- 부모 클래스에서 "이 메서드를 정의해라"라고 자식 클래스에게 알려주는 개념에 가까움
	- **강제성을 띔 : 자식 클래스는 반드시 이 메서드를 정의**해야 함
	- `abstract` 메서드가 선언된 클래스는, 클래스 자체도 `abstract` 선언이 되어야 함

- 구현 과정에서는 `virtual`이 훨씬 편리한 느낌이다. 빈 함수로 정의하면 된다. `abstract`가 강제성을 띄기 때문.
```cs
// 부모 클래스에서의 정의
public abstract void Activate(Operator op);
public virtual void PerformSkillAction(Operator op) { }
```

