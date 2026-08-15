
- 이전 : [[Effective Csharp - 1. Csharp 언어 요소]]
- 다음 : [[Effective Csharp - 3. 제네릭 활용]]

---

1. [[#NET 리소스 관리에 대한 이해|NET 리소스 관리에 대한 이해]]
2. [[#할당 구문보다 멤버 초기화 구문이 좋다|할당 구문보다 멤버 초기화 구문이 좋다]]
3. [[#정적 클래스 멤버를 올바르게 초기화하라|정적 클래스 멤버를 올바르게 초기화하라]]
4. [[#초기화 코드의 중복을 최소화하라|초기화 코드의 중복을 최소화하라]]
5. [[#불필요한 객체를 만들지 마라|불필요한 객체를 만들지 마라]]
6. [[#생성자 내에서는 절대로 가상 함수를 호출하지 말라|생성자 내에서는 절대로 가상 함수를 호출하지 말라]]
7. [[#표준 Dispose 패턴을 구현하라|표준 Dispose 패턴을 구현하라]]

---

## NET 리소스 관리에 대한 이해
- 가비지 컬렉터의 동작 원리를 이해해야 메모리 누수를 막고 관리되지 않는 리소스를 제대로 정리할 수 있다.

- 유니티
	- **유니티 오브젝트`GameObject, Texture, Mesh 등`는 C# 관리 객체 껍데기 + `C++` 네이티브 리소스로 이뤄진 이중 구조**이다.
		- `Destroy(obj)`를 호출해도 네이티브 리소스는 즉시 해제되나, C# 래퍼 객체는 GC가 따로 수거해야 한다. 그래서 `Destroy()` 후에도 `obj == null` 비교가 유니티에서만 특별하게 동작되도록 오버로딩되어 있다.
	- `Texture2D`, `Mesh`, `RenderTexture` 등 명시적으로 `Destroy()`나 `Resources.UnloadAsset()`을 호출하지 않으면 메모리에 계속 남는 리소스들이 있다. 
		- 이들은 GC가 알아서 치워주는 일반 C# 객체와는 다르다.

```cs
void ReplaceTexture(Texture2D newTex)
{
    // 나쁜 예: 기존 텍스처를 그냥 참조만 바꾸면 네이티브 메모리 누수
    _material.mainTexture = newTex;

    // 좋은 예: 기존 텍스처를 명시적으로 해제
    var old = _material.mainTexture as Texture2D;
    _material.mainTexture = newTex;
    if (old != null)
    {
        Destroy(old); // 네이티브 리소스 즉시 해제
    }
}
```


>[!question]
>- 이참에 `Destroy(obj)`가 어떻게 동작하는지 복습해도 좋을 것 같다.
>	- 내가 알고 있기론 `Destroy(obj)`를 하더라도 `null`체크를 해도 `null`이 아닌 걸로 나오는 걸로 알고 있음. 저 오버로딩의 의미라고 해야 할까?
>- 예제 코드의 나쁜 예도 궁금하다. 기존 텍스쳐가 참조를 잃는데, 유니티의 GC는 참조를 잃은 객체를 수거하는 걸로 알고 있는데 기존 텍스쳐를 수거하지 않는다는 의미인가?


## 할당 구문보다 멤버 초기화 구문이 좋다
- 변수 선언과 동시에 초기화하면 여러 생성자에서 초기화 코드가 누락되는 실수를 방지할 수 있다.

- 유니티
	- **`MonoBehaviour`는 생성자를 직접 호출할 수 없다. `Awake()`가 생성자 역할을 하는 경우가 많다.** 
		- `new MyComponent()`를 본 적이 없죠?
	- **필드 선언부에서 기본값**을 미리 잡아두면 `Awake()` 실행 전에 안전한 기본값이 보장된다.

```cs
public class Inventory : MonoBehaviour
{
    // 좋은 예: 필드 선언과 동시에 초기화 (Awake 실행 전에도 null이 아님)
    private List<Item> _items = new List<Item>();
    private int _maxSlots = 20;

    // 나쁜 예: Awake에서만 초기화하면, 실행 순서에 따라
    // 다른 스크립트가 Awake보다 먼저 _items에 접근할 경우 NullReferenceException
    void Awake()
    {
        // _items = new List<Item>(); // 여기서만 하면 위험
    }
}
```

## 정적 클래스 멤버를 올바르게 초기화하라
- 정적 생성자를 사용하면 정적 멤버가 사용되기 전에 단 한 번만 스레드가 안전하게 초기화됨을 보장받을 수 있다.

- 유니티
	- 유니티에서는 `MonoBehaviour` 기반 싱글턴이 더 흔하다. 
	- 순수 데이터 / 유틸 클래스일 경우에만 정적 생성자가 유용하다.
		- 정적 생성자 내부에서 유니티 API를 호출하면 메인 스레드 / 씬 로드 타이밍 문제가 발생할 수 있으니, 순수 데이터 초기화 용도로만 쓰는 게 안전하다.

```cs
public static class GameConfig
{
    public static readonly Dictionary<string, float> BalanceTable;

    // 정적 생성자: GameConfig가 최초로 사용되는 시점에 단 한 번, 스레드가 안전하게 실행됨
    static GameConfig()
    {
        BalanceTable = new Dictionary<string, float>
        {
            { "PlayerHP", 100f },
            { "EnemyDamage", 15f }
        };
    }
}

// 사용: 최초 접근 시점에 자동으로 static 생성자가 실행됨
float hp = GameConfig.BalanceTable["PlayerHP"];
```

> **정적 클래스의 생성자는 최초로 접근할 때 동작한다**는 점만 유의해두면 될 듯.


## 초기화 코드의 중복을 최소화하라
- `this()` 생성자 체인 호출을 활용해 초기화 로직을 한 곳으로 모으면 유지보수가 쉬워지고 오류를 줄일 수 있다.

- 유니티
	- 마찬가지로 순수 C# 클래스에서 적용할 수 있는 이야기.
	- `SO`도 생성자 대신 `OnEnable`이나 팩토리 메서드 `CreateInstance`를 쓰는 게 관례이다.

```cs
public class EnemyData
{
    public string Name { get; }
    public int Hp { get; }
    public int Damage { get; }

    // 모든 초기화 로직이 여기 한 곳에 모임
    public EnemyData(string name, int hp, int damage)
    {
        Name = name;
        Hp = hp;
        Damage = damage;
    }

    // this()로 체인 -> 중복 없이 기본값만 지정
    public EnemyData(string name) : this(name, hp: 50, damage: 5) { }
    public EnemyData() : this("Unnamed") { }
}
```

>[!question]
>1. `SO`에서 `OnEnable`을 쓴다고?? 랑 `CreateInstance`가 뭐지??가 다 궁금함.
>2. 예제에서 `this`를 쓰는 방식도 궁금함.



## 불필요한 객체를 만들지 마라
- 짧은 생명주기를 가진 참조 객체를 과도하게 생성하면 가비지 컬렉터에 과부하를 주어 앱 성능이 저하된다.

- 유니티
	- `Update()/FixedUpdate()`에서의 할당이 유니티 성능 최적화의 8할이다. 
	- 아래는 주의해야 할 패턴들.
		- `new Vector3(...)`은 구조체라서 괜찮음.
		- **`new List<T>`, `new WaitForSeconds(1f)`, 람다 클로저`() => {...}` 등은 모두 힙 할당 요소**이다.
		- `foreach`도 일부 컬렉션에서 이터레이터 박싱이 발생할 수 있다.
		- 문자열 연결`+`도 매 프레임 진행하면 누적 할당이다.

```cs
public class Shooter : MonoBehaviour
{
    // 좋은 예: 캐싱해서 재사용
    private WaitForSeconds _fireDelay = new WaitForSeconds(0.2f);
    private List<GameObject> _bulletPool = new List<GameObject>();

    IEnumerator FireRoutine()
    {
        while (true)
        {
            // 나쁜 예: 매번 새 WaitForSeconds 객체 생성 (매 루프 GC 대상)
            // yield return new WaitForSeconds(0.2f);

            // 좋은 예: 캐싱된 인스턴스 재사용
            yield return _fireDelay;
            Fire();
        }
    }

    void Fire()
    {
        // 나쁜 예: 매번 새 오브젝트 생성/파괴
        // var bullet = Instantiate(bulletPrefab);

        // 좋은 예: 오브젝트 풀에서 재사용
        var bullet = GetFromPool();
        bullet.SetActive(true);
    }

    GameObject GetFromPool() => _bulletPool.Count > 0 ? _bulletPool[0] : null;
}
```

>  **`WaitForSeconds()`가 캐싱해서 쓸 수 있는 거였음??? 충격.** 



## 생성자 내에서는 절대로 가상 함수를 호출하지 말라
- 베이스 클래스의 생성자 호출 시점에는 파생 클래스의 객체가 완전히 초기화되지 않았기 때문에, 가상 메서드를 이 때 부르면 예기치 않은 오류가 발생한다.


## 표준 Dispose 패턴을 구현하라
- 관리되지 않는 리소르를 다루는 타입은 `IDisposable`을 상속하고 `Dispose` 패턴을 구현해서 리소스 해제를 보장해야 한다.

