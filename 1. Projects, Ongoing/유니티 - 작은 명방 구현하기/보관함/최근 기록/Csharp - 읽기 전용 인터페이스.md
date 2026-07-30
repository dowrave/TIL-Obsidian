
## 문제 상황
- 유닛이 피해를 입는 상황을 가정해보자.
- `UnitEntity`가 생성될 때 체력 시스템을 담당하는 `HealthSystem`도 생성된다
- `HealthSystem`의 메서드에 접근하기 위해 `HealthSystem`의 메서드는 `public`으로 공개된다. `ProcessDamage()`
- 이외에도 `HealthSystem`의 값을 외부에서 읽어야 하는 경우가 있어서 `UnitEntity`에서 `HealthSystem`을 `public`으로 공개했다.
- `UnitEntity` 자체에도 `TakeDamage()`라는 퍼블릭 메서드가 있다고 가정한다. `UnitEntity`는 대미지 값 처리 외에도 이펙트, 사운드 재생 등을 모두 포함한다.

따라서 외부에서 유닛에게 대미지를 입히는 상황이라면 접근할 수 있는 곳이 두 곳이 된다. `UnitEntity.HealthSystem.ProcessDamage()`, `UnitEntity.TakeDamage()`. 이는 혼동을 낳을 수 있다.

개발자가 원하는 건 `UnitEntity.TakeDamage()`로만 접근할 수 있게 하는 것이다. 약속을 정해둘 수 있겠지만, 실수를 100% 방지할 수는 없는 상황.

## 읽기 전용 인터페이스
- 기존 클래스에서 게터 프로퍼티 / 메서드만 인터페이스로 뺀다.
```cs
// 읽기 전용 인터페이스
public interface IReadOnlyHealth
{
    float CurrentHealth { get; }
    float MaxHealth { get; }
    event Action<float, float, float> OnHealthChanged;
    
    // 변경 메서드(ProcessDamage 등)는 여기에 포함하지 않음!
}
```

- 그리고 원본 클래스에 이 인터페이스를 `구현Implementation`시킨다.
```cs
// 인터페이스가 적용되는 원본 클래스
public class HealthSystem : IReadOnlyHealth
{
    public float CurrentHealth { get; private set; }
    public float MaxHealth { get; private set; }
    // ... 나머지 구현
    
    // 이 메서드는 public이지만 인터페이스에는 없으므로,
    // 인터페이스로 접근하는 사람은 이 메서드의 존재를 모름
    public float ProcessDamage(AttackSource source) { /* ... */ }
}
```

- 마지막으로 `UnitEntity`에는 
	1. 자신만이 접근할 수 있는 `private`으로 `HealthSystem`을
	2. 외부에서 접근할 수 있는 `public`으로 `IReadOnlyHealth`을 구현한다.
```cs
public class UnitEntity: MonoBehaviour
{
	// UnitEntity만이 알고 있는 실제 알맹이
	private HealthSystem _healthSystem;
	
	// 외부에 보여주는 껍데기
	public IReadOnlyHealth Health => _healthSystem;
	
	// 이제 외부 사용자는 실제 대미지 계산 로직에 접근할 방법이 없다.
	// UnitEntity에 있는 TakeDamage만 실행시킬 수 있음.
	public void TakeDamage(AttackSource source)
	{
		_healthSystem.ProcessDamage(source);
	}
}
```

이렇게 구현하면 `UnitEntity`을 제외한 외부에서는 `HealthSystem.ProcessDamage()`에 접근할 방법이 없어지게 된다.

