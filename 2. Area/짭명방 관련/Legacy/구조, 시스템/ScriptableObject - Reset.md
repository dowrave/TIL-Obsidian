```cs
public bool autoRecover; // SP 자동회복 여부
public bool autoActivate; // 자동발동 여부
public bool modifiesAttackAction; // 기본 공격이 다르게 나가는 스킬일 때 true
```
> 이 필드들은 BaseSkill에 있고 디폴트로 `false`를 지정하지만, 자식 클래스들에서는 활성화되어야 할 수도 있음
> 한편 같은 스킬 구성을 띄어도 스킬을 어떻게 구현하느냐에 따라 달라질 수 있음. 이거는 어떤 걸 구현하고 싶냐에 따라 항상 달라지는 요소임
> - 예를 들면 똑같은 쉴드 스킬이어도 공격 x번이 동작한 후에 자동으로 동작하는 쉴드가 있을 수도 있고, 가만히 기다렸다가 SP가 꽉 차면 유저가 원하는 타이밍에 쉴드를 동작시킬 수도 있음
> - 그래서 저 값들을 초기화는 해주되 개발자가 자유롭게 변경이 가능하도록 구성해야 하는데, 아래처럼 설정하면 된단다.

```cs
protected virtual void SetDefaults()
{
	autoRecover = false;
	autoActivate = false;
	modifiesAttackAction = false; 
}

protected void Reset()
{
	SetDefaults();
}
```
> `Reset`은 `ScriptableObject`를 만들 때에만 1번 동작하는 메서드이다. `SetDefaults`만 자식에서 재정의해주면 됨.
> **다형성에 의해 자식에서 실행되는 `Reset`은 자식에서 오버라이드된 `SetDefaults`가 있다면 그 함수를 실행시킴.** 이전에도 다뤘던 기억이 있지만, 헷갈릴 수 있으니 염두에 두자.
