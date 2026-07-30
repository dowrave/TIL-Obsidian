- **클래스 단위로 이벤트를 짤 수 있다.**
- 예를 들면 `Enemy`가 죽을 때마다 모든 `Operator`에게 알리는 이벤트를 만들 수 있다.
- **`static event`가 그런 역할**을 한다.

- 예시) Enemy가 죽을 때마다 모든 Operator에게 알리기

1. `static event`를 `enemy`에 만듦
```cs
public class Enemy 
{
	// 모든 Enemy가 공유하는 사망 이벤트
	public static event System.Action<Enemy> OnEnemyDied;

	// ...
	
	protected override void Die() 
	{
		OnEnemyDied?.Invoke(this);
	}
}
```

2. `operator`에서 해당 이벤트를 구독하도록 함
```cs
public override void Deploy(Vector3 position)
{
	base.Deploy(position);
	
	// 이벤트 구독
	Enemy.OnEnemyDied += HandleEnemyDeath; // 클래스로 접근하는 것에 주목
}

protected override void Die() 
{
	Enemy.OnEnemyDied -= HandleEnemyDeath;
}
```

만약 전역적인 단위에서 체크가 필요하다면 `Awake`에서 클래스 단위의 이벤트를 등록하고 `OnDestroy`에서 해제하는 방법도 괜찮다. 씬 전환 상황 등에서 해제.