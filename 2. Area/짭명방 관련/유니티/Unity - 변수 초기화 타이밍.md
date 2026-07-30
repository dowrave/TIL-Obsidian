#Unity

```cs
private List<Enemy> blockedEnemies = new List<Enemy>();
```
> 변수를 선언하면서 초기화하는 경우, 이 `blockedEnemies`는 `Awake()` 이전에 초기화된다.

- 만약 조건에 따라 다르게 초기화하는 게 필요하다면, `Awake()` 메서드 등에서 이를 구현해놓는 게 좋다.