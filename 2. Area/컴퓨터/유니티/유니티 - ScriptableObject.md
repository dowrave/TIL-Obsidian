
1. [[#구조는 다른 것과 동일함|구조는 다른 것과 동일함]]
2. [[#다른 객체와 중요한 차이점|다른 객체와 중요한 차이점]]
3. [[#전략 객체로서의 활용|전략 객체로서의 활용]]

## 구조는 다른 것과 동일함

- 기본 구조는 결국 `UnityEngine.Object`를 상속한 타입이다. 이런 측면에서 보면 `MonoBehaviour`, `Texture2D`, `GameObject` 등과 같은 계열이다.

```
디스크(.asset 파일, YAML/바이너리로 직렬화된 데이터)
        │
        │  로드(씬 로드, Resources.Load, 직접 참조 등)
        ▼
메모리(C# 관리 객체 + 네이티브 객체) ← 여기서부터는 그냥 "객체"
        │
        │  필드에 대입
        ▼
당신의 스크립트가 들고 있는 레퍼런스
```

**디스크에 있을 때는 데이터 덩어리지만, 참조로 메모리에 로드된 후부터는 일반 C# 객체와 동일하게 동작**한다. `OnEnable()`이 호출되고, 필드에 값이 채워지고, 스크립트가 레퍼런스를 필드에 들고 있으면 해당 객체를 가리키는 개념으로, `GameObject` 필드에 씬의 오브젝트를 드래그해서 연결하는 것과 원리적으로 똑같다.


```cs
public class Player : MonoBehaviour
{
    [SerializeField] private EnemyDataSO _enemyData; // 인스펙터에서 애셋을 드래그해서 연결

    void Start()
    {
        // 이 시점엔 _enemyData가 이미 메모리에 로드되어 있는 "그냥 객체"
        Debug.Log(_enemyData.hp);
    }
}
```

이런 식으로 필드에 연결되면, 유니티는 씬이나 프리팹을 로드할 때 참조된 SO 애셋도 자동으로 함께 메모리에 로드해준다. 

## 다른 객체와 중요한 차이점

>[!note]
>- **SO는 기본적으로 공유된 하나의 인스턴스**이다.

```cs
public class EnemyDataSO : ScriptableObject
{
    public int hp = 100;
}

public class EnemyA : MonoBehaviour
{
    [SerializeField] private EnemyDataSO _data; // 인스펙터에서 같은 애셋(예: "GoblinData.asset")을 연결
}

public class EnemyB : MonoBehaviour
{
    [SerializeField] private EnemyDataSO _data; // 똑같이 "GoblinData.asset"을 연결
}

void TakeDamage(int amount)
{
    _data.hp -= amount; // 나쁜 예: 이 SO를 참조하는 모든 적의 hp가 같이 깎임!
}
```
- 만약 `EnemyA`와 `EnemyB`의 현재 체력이 `EnemyDataSO`의 체력값과 동일하다면, A가 공격을 받든 B가 공격을 받든 `EnemyDataSO`의 체력을 참조하는 모든 객체가 체력을 공유하는 개념이 된다. 

## 전략 객체로서의 활용
- 일반적으로 SO를 쓰는 방식.
- **자신은 애셋으로만 존재하고, 런타임 중에 변하는 상태 필드를 두지 않는다.**

