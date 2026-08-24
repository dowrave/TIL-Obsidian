#Csharp #EffectiveCsharp

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

### `Destroy(obj)`의 동작

```cs
Texture2D tex = _material.mainTexture as Texture2D;
Destroy(tex);

Debug.Log(tex == null);              // true <- 오버로딩된 == 연산자 결과
Debug.Log(ReferenceEquals(tex, null)); // false <- 진짜 C# 레퍼런스는 아직 살아있음
```
**`Destroy(obj)` 후, `obj == null`을 하면 `true`로 나오는데, 이것 자체가 `==` 연산자의 오버로딩**이다. 
- 즉 **실제로 `obj`는 `null`이 아니**라는 것임.

`Destroy(obj)`는 즉시 객체를 없애지 않고, 네이티브 C++ 쪽 리소스를 이번 프레임이 끝날 때 파괴하도록 예약한다. 그런데 C# 쪽의 `obj` 변수는 여전히 진짜 객체를 가리키는 `관리 객체Managed Wrapper`다. 이 객체는 GC가 나중에 따로 치우는 것으로, `Destroy()`호출과 동시에 사라지는 것이 아니다.

그래서 **유니티는** `UnityEngine.Object`에 **`==`, `!=` 연산자를 오버로딩해서 "C# 레퍼런스가 null인가?" 여부가 아니라 "네이티브 객체가 파괴되었는가?"를 체크**하도록 만들었다. 

- 네이티브 객체가 파괴됨 : `obj == null`은 `true`를 반환, 마치 `null`인 것 처럼 동작.
- 그러나 C# 레벨에서 `obj`는 여전히 존재하는 객체(껍데기)를 참조하는 중이라서 `ReferenceEquals(obj, null)`은 `false`라는 것.

이를 `fake null`이라고 부른다. 이 오버로딩 때문에 `obj == null` 체크는 내부적으로 네이티브 플러그인을 호출하는 살짝 비용이 있는 연산이라서, 매 프레임 수천 번 반복되는 연산에서는 `if (obj)`으로 최소화하는 걸 권장한다. (암시적 bool 변환, 오버로딩된 요소)

### 리소스 메모리 관리 관련
```cs
// 나쁜 예: 기존 텍스처를 그냥 참조만 바꾸면 네이티브 메모리 누수
_material.mainTexture = newTex;
```

>[!question]
> `mainTexture`가 바뀌었을 때, 기존의 `mainTexture`를 GC가 정리하지 않는다는 부분이 의아했다.

위 코드에서 잃어버리는 건 C# 관리 객체인 작은 wrapper에 대한 레퍼런스로, 이는 `.NET GC`가 알아서 수거한다. 

하지만 **GPU에 올라가 있는 실제 텍스쳐 데이터(픽셀 데이터, VRAM)는 `.NET GC`의 관리 범위 밖에 있는 비관리 네이티브 메모리**다. 

C#의 GC는 다음 2가지 방법으로만 해제된다. (유니티의 GC는 C#의 GC를 자체 커스터마이징한 것.)
1. `Destroy(oldTexture)`을 명시적으로 호출
2. `REsources.UnloadUnusedAssets()`이나 씬 전환 시 자동으로 실행되는 정리 루틴.

따라서, C# 레퍼런스를 잃는 것과 네이티브 리소스가 해제되는 것은 유니티에서 별개의 이벤트로, 레퍼런스를 잃은 채 `Destroy()`를 부르지 않는다면 C# 쪽의 껍데기는 GC가 치우지만 VRAM의 텍스쳐 데이터는 계속 남아있는 네이티브 메모리 누수가 발생한다. 




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

### SO의 OnEnable, CreateInstance, this()

>[!question]
>1. `SO`에서 `OnEnable`을 쓴다고?? 랑 `CreateInstance`가 뭐지??가 다 궁금함.
>2. 예제에서 `this`를 쓰는 방식도 궁금함.

- 일단 일반적으로 써오던 패턴이랑 다르다는 점은 유의하고 보셈 
	- 보통은 데이터의 저장 창고 같은 개념으로 사용해왔으니까, 런타임 중에 SO를 생성한다는 개념이 낯설긴 하다.

```cs
public class EnemyDataSO : ScriptableObject
{
    public string enemyName;
    public int hp;

    // 생성자 대신 OnEnable을 초기화 지점으로 사용
    void OnEnable()
    {
        if (string.IsNullOrEmpty(enemyName))
            enemyName = "Unnamed Enemy";
    }
}

// 사용
var data = ScriptableObject.CreateInstance<EnemyDataSO>();
data.hp = 100;
```

- `CreateInstance()`
	- SO는 `new MySO()`로 직접 생성하면 안된다. 공식 문서에서도 금지한 것으로, 유니티 엔진 내부에 제대로 등록되지 않아서 오류가 발생한다.
	- 대신 유니티가 제공하는 팩토리 메서드 `ScriptableObject.CreateInstance<T>()`를 사용한다.

- `OnEnable()`
	- `SO`는 에셋 파일로 직렬화된다. 에디터에서 스크립트를 수정하면 도메인 리로드가 일어나면서 C# 객체가 다시 만들어지고 데이터가 역직렬화된다.
	- C# 생성자는 항상 예측 가능하게 호출된다고 보장할 수 없지만, **`OnEnable()`은 로드 / 생성 / 도메인 리로드 시점마다 유니티가 확실하게 호출해주는 콜백이라서 초기화 시점으로 안전하게 쓸 수 있다.** 

- `this()` 체인 생성자
	- 일반 C# 클래스에서 생성자 하나가 같은 클래스의 다른 생성자를 먼저 호출하도록 연결하는 문법. 
	- 내 경우 짭명방 프로젝트에서 컨트롤러 상속할 때 썼던 걸로 기억함.
```cs
public class EnemyData
{
    public string Name { get; }
    public int Hp { get; }
    public int Damage { get; }

    // 1. 이게 진짜 초기화 로직이 있는 "메인" 생성자
    public EnemyData(string name, int hp, int damage)
    {
        Name = name;
        Hp = hp;
        Damage = damage;
    }

    // 2. name만 받으면 -> 1번 생성자를 hp=50, damage=5로 호출
    public EnemyData(string name) : this(name, hp: 50, damage: 5) { }

    // 3. 매개변수 없으면 -> 2번 생성자를 "Unnamed"로 호출 -> 결국 1번까지 이어짐
    public EnemyData() { }
}
```
> 실제 필드 대입 로직은 1번에만 존재한다. 
> 만약 `this()`가 없었다면 각 생성자마다 필드를 복붙해야 할 거고, 필드가 하나 추가될 때마다 모든 생성자를 다 고쳐야 할 것이다. 그 위험을 원천 차단하는 역할.



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

- 유니티
	- 비슷한 역할을 하는 `Awake()`에서도 비슷한 함정에 빠질 수 있다. 
	- 베이스 클래스에서 오버라이드된 파생 클래스의 메서드를 호출하면, 파생 클래스의 `Awake()`가 아직 초기화되지 않은 필드를 건드릴 수 있다. 
```cs
public class Character : MonoBehaviour
{
    protected virtual void Awake()
    {
        // 나쁜 예: 파생 클래스가 override한 Initialize()를 여기서 호출
        // 이 시점엔 BossCharacter의 필드(_phase 등)가 아직 세팅 안 됐을 수 있음
        Initialize();
    }

    protected virtual void Initialize()
    {
        Debug.Log("기본 초기화");
    }
}

public class BossCharacter : Character
{
    private int _phase = 3; // 필드 초기화는 base.Awake()보다 먼저 실행되긴 하지만,
                             // 복잡한 초기화 체인에서는 순서를 확신하기 어려움

    protected override void Initialize()
    {
        // _phase를 여기서 쓰는 게 안전해 보여도, 호출 시점에 따라 위험할 수 있음
        Debug.Log($"보스 초기화, 페이즈: {_phase}");
    }
}

// 더 안전한 대안: 각자 자신의 Awake에서 독립적으로 초기화하고,
// 부모->자식 순서가 꼭 필요하면 별도의 명시적 Init() 메서드를 매니저가 순서대로 호출
```
> 이거는 **템플릿 메서드 패턴 쓸 때 유의할 점**으로 보인다. 
> 근데 또 뜯어보면 이 말이 맞나? 싶기도 해.


### 템플릿 메서드 관련

- 안전한 케이스
	- 진짜 C# 생성자와 달리, **`MonoBehaviour`의 필드 초기화 구문`private int _phase = 3;`은 `Awake()` 호출 훨씬 이전**에, 유니티가 오브젝트를 생성하는 시점에 끝나 있다. 
	- 그렇기 때문에 `base.Awake()`에서 파생 클래스가 `override`한 가상 메서드를 호출하더라도, 필드 초기화 값만 사용한다면 안전하다.
```cs
public abstract class Character : MonoBehaviour
{
    protected virtual void Awake()
    {
        Debug.Log("공통 초기화 시작");
        Setup(); // 템플릿 메서드 패턴: 하위 클래스가 재정의
        Debug.Log("공통 초기화 끝");
    }

    protected abstract void Setup();
}

public class BossCharacter : Character
{
    // 필드 초기화는 Awake보다 먼저 완료됨 -> 안전
    private int _phase = 3;

    protected override void Setup()
    {
        Debug.Log($"보스 셋업, 페이즈: {_phase}"); // 안전: 이미 3으로 세팅됨
    }
}
```

- 위험한 케이스
	- `base.Awake()`가 호출하는 가상 메서드가, 파생 클래스의 `Awake()` 본문 안에서 실행되는 코드에 의존할 때.
```cs
public class BossCharacter : Character
{
    private List<Attack> _attackPatterns;

    protected override void Awake()
    {
        // base.Awake()를 먼저 부르면, base.Awake() 내부에서 Setup()이 호출되는데
        // 이 시점엔 아직 아래 _attackPatterns 초기화가 실행 전임 -> 위험!
        base.Awake();

        _attackPatterns = LoadAttackPatterns(); // 이 줄이 Setup()보다 늦게 실행됨
    }

    protected override void Setup()
    {
        // _attackPatterns가 아직 null인 상태에서 호출될 수 있음
        Debug.Log(_attackPatterns.Count); // NullReferenceException 위험
    }
}
```
> - 사실 이 예제는 `_attackPatterns`가 필드에서 초기화되면 되는 거 아닌가? 싶기는 한데..
> - `new List()`로 초기화해도 큰 의미는 없다. `_attackPatterns`에 의미 있는 값으로 초기화된 게 아니기 때문이다. 

이야기의 본질은 "**`Setup()`이 의존하는 모든 것이 `Setup()` 호출 이전에 확정되도록 실행 순서를 재배치하자**"에 가깝다. 호출되는 시점에 의존성이 이미 준비되어 있는가?에 관한 이야기. 


## 표준 Dispose 패턴을 구현하라
- 관리되지 않는 리소스를 다루는 타입은 `IDisposable`을 상속하고 `Dispose` 패턴을 구현해서 리소스 해제를 보장해야 한다.

- 유니티
	- `OnDestroy()`가 그 역할을 수행한다. 이벤트 구독 해제, 코루틴 정지, 네이티브 리소스 해제 등을 여기서 수행하면 됨.
		- 자주 실수하는 것) 정적 이벤트의 구독 해제는 `OnDestroy()`에서 해주자.
	- `IDisposable`은 순수 C# 클래스를 사용하는 경우에는 그대로 적용된다. 예를 들면 네트워크 클라이언트 래퍼, 파일 스트리밍을 다루는 유틸 등.

- 순수 C# 클래스 예시 코드
```cs
// 순수 C# 클래스에서 표준 Dispose 패턴
public class NetworkClient : IDisposable
{
    private Socket _socket;
    private bool _disposed = false;

    public NetworkClient()
    {
        _socket = new Socket(/* ... */);
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing)
        {
            _socket?.Close(); // 관리 리소스 정리
        }
        _disposed = true;
    }

    ~NetworkClient()
    {
        Dispose(false);
    }
}
```

- 유니티 예시 코드
```cs
// 유니티 MonoBehaviour에서의 대응: OnDestroy
public class ScoreManager : MonoBehaviour
{
    void OnEnable()
    {
        GameEvents.OnEnemyKilled += HandleEnemyKilled; // 정적 이벤트 구독
    }

    void OnDestroy()
    {
        // 반드시 해제. 안 하면 오브젝트가 파괴돼도 GameEvents가 참조를 들고 있어 누수
        GameEvents.OnEnemyKilled -= HandleEnemyKilled;
    }

    void HandleEnemyKilled(Enemy e) { /* ... */ }
}
```
