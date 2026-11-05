#Csharp #EffectiveCsharp

1. [[#반드시 필요한 제약 조건만 설정하라|반드시 필요한 제약 조건만 설정하라]]
2. [[#런타임에 타입을 확인해 최적의 알고리즘을 사용하라|런타임에 타입을 확인해 최적의 알고리즘을 사용하라]]
3. [[#IComparable과 IComparer을 사용해서 객체의 선후 관계를 정리하라|IComparable과 IComparer을 사용해서 객체의 선후 관계를 정리하라]]
	1. [[#IComparable과 IComparer을 사용해서 객체의 선후 관계를 정리하라#IComparable - "나 자신이 다른 것과 비교될 수 있다"|IComparable - "나 자신이 다른 것과 비교될 수 있다"]]
	2. [[#IComparable과 IComparer을 사용해서 객체의 선후 관계를 정리하라#IComparer - "내가 아니라 제3자가 비교 기준을 제공한다"|IComparer - "내가 아니라 제3자가 비교 기준을 제공한다"]]
4. [[#타입 매개변수가 `IDisposable`을 구현할 경우를 대비해 제네릭 클래스를 작성하라|타입 매개변수가 `IDisposable`을 구현할 경우를 대비해 제네릭 클래스를 작성하라]]
	1. [[#타입 매개변수가 `IDisposable`을 구현할 경우를 대비해 제네릭 클래스를 작성하라#IDisposable - "나는 명시적으로 정리해야 할 리소스를 갖고 있다"|IDisposable - "나는 명시적으로 정리해야 할 리소스를 갖고 있다"]]
5. [[#공변성과 반공변성을 지원하라|공변성과 반공변성을 지원하라]]
	1. [[#공변성과 반공변성을 지원하라#공변성, 반공변성|공변성, 반공변성]]
	2. [[#공변성과 반공변성을 지원하라#BCL(Base Class Library)|BCL(Base Class Library)]]
6. [[#타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 이용하라|타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 이용하라]]
7. [[#베이스 클래스나 인터페이스에 대해 제네릭을 특화하지 말라|베이스 클래스나 인터페이스에 대해 제네릭을 특화하지 말라]]
8. [[#타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제네릭 메서드를 정의하라|타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제네릭 메서드를 정의하라]]
9. [[#제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라|제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라]]
10. [[#인터페이스는 간략히 정의하고 확장 메서드로 기능을 더하라|인터페이스는 간략히 정의하고 확장 메서드로 기능을 더하라]]
11. [[#확장 메서드를 이용해 구체화된 제네릭 타입을 개선하라|확장 메서드를 이용해 구체화된 제네릭 타입을 개선하라]]

## 반드시 필요한 제약 조건만 설정하라
- 제네릭 매개변수에 과도한 제약을 걸면 재사용성이 떨어진다. 컴파일러가 요구하는 최소한의 제약만 추가해야 한다. 

- 유니티
	- 오브젝트 풀, 이벤트 시스템 등 범용 유틸리티를 제네릭으로 만들 때 흔히 실수하는 부분이다. 
	- "혹시 몰라서" `where T : MonoBehaviour, new()` 처럼 필요 이상으로 제약을 걸 경우 나중에 순수 데이터 클래스에는 사용할 수 없게 된다.

```cs
// 나쁜 예: 과도한 제약 (MonoBehaviour일 필요가 없는데도 강제)
public class ObjectPool<T> where T : MonoBehaviour, new() { }

// 좋은 예: Component만 있으면 충분 (Instantiate/GetComponent에 필요한 최소 조건)
public class ObjectPool<T> where T : Component
{
    public T Rent() { /* ... */ return null; }
}
```

## 런타임에 타입을 확인해 최적의 알고리즘을 사용하라
- 제네릭 타입은 일반적인 로직을 제공하지만, 런타임에 특정 타입(`is` 연산자 등)을 확인해 그에 맞는 더 빠른 알고리즘으로 분기할 수 있다.

- 유니티
	- 값 타입`struct`과 참조 타입을 섞어 다루는 제네릭 유틸리티에서 값 타입일 땐 박싱 없이 처리하는 분기를 추가하는 식으로 응용한다.
```cs
public static bool AreEqual<T>(T a, T b)
{
    // T가 IEquatable<T>를 구현하면 박싱 없는 비교 (더 빠름)
    if (a is IEquatable<T> equatableA)
    {
        return equatableA.Equals(b);
    }
    
    // 그렇지 않으면 일반 Equals (박싱 가능성 있음)
    return Equals(a, b);
}
```

## IComparable과 IComparer을 사용해서 객체의 선후 관계를 정리하라
- 컬렉션 정렬/검색을 위해 표준 비교 인터페이스를 구현하라.

- 유니티
	- 적 우선순위 정렬, 거리순 정렬 등에서 매우 자주 쓴다.
	- `List<T>.Sort()`가 이 인터페이스들을 쓴다.

```cs
public class Enemy : IComparable<Enemy>
{
    public float DistanceToPlayer;

    // 기본 정렬 기준 (거리순)
    public int CompareTo(Enemy other) => DistanceToPlayer.CompareTo(other.DistanceToPlayer);
}

// 기본 기준과 다른 정렬이 필요할 때는 별도 IComparer 구현
public class ByThreatComparer : IComparer<Enemy>
{
    public int Compare(Enemy a, Enemy b) => b.ThreatLevel.CompareTo(a.ThreatLevel); // 내림차순
}

// 사용
enemies.Sort(); // IComparable 사용 (거리순)
enemies.Sort(new ByThreatComparer()); // IComparer 사용 (위협도순)
```

### IComparable - "나 자신이 다른 것과 비교될 수 있다"

- 기본 정렬 메서드가 이 기준을 쓴다.
```cs
public interface IComparable<T>
{
    int CompareTo(T other);
}
```
- 반환값 규칙
	- 음수 : `this < other`
	- 0 : `this == other`
	- 양수 : `this > other`

```cs
public class Enemy : IComparable<Enemy>
{
    public float Distance;

    public int CompareTo(Enemy other)
    {
        return Distance.CompareTo(other.Distance); // float도 CompareTo를 지원
    }
}

var list = new List<Enemy> { /* ... */ };
list.Sort(); // 거리 오름차순 정렬 (CompareTo 기준)
```
- 클래스에 기본적인, 단 하나뿐인 정렬 기준이 있을 때 쓴다. 

### IComparer - "내가 아니라 제3자가 비교 기준을 제공한다"
- 완전히 별개의 클래스가 두 객체를 비교하는 기준을 제공한다.
```cs
public interface IComparer<T>
{
    int Compare(T x, T y);
}
```
- 이게 왜 필요함?
	- `Enemy`는 "거리"라는 기본 정렬 기준이 있지만, 상황에 따라 "위협도"라든가 "알파벳"이라든가 다른 기준을 넣고 싶을 때가 있다.
	- `IComparable`은 클래스 당 1개만 만들 수 있지만, `IComparer`는 여러 개 만들 수 있다.

```cs
public class ByThreatComparer : IComparer<Enemy>
{
    public int Compare(Enemy x, Enemy y) => y.ThreatLevel.CompareTo(x.ThreatLevel); // 내림차순
}

public class ByNameComparer : IComparer<Enemy>
{
    public int Compare(Enemy x, Enemy y) => string.Compare(x.Name, y.Name);
}

// 상황별로 골라서 사용
list.Sort();                        // 기본 기준(IComparable, 거리순)
list.Sort(new ByThreatComparer());  // 위협도순
list.Sort(new ByNameComparer());    // 이름순
```


## 타입 매개변수가 `IDisposable`을 구현할 경우를 대비해 제네릭 클래스를 작성하라
- 제네릭 클래스 내부에서 `T`의 인스턴스를 직접 생성했다면, `T`가 `IDisposable`일 가능성을 대비해 해제 로직을 넣어야 한다.

- 유니티
	- 제네릭 오브젝트 풀을 만들 때, 풀에 담긴 객체가 네트워크 연결이나 파일 핸들 같은 리소스를 들고 있을 수 있다. 풀을 비울 때 이를 놓치면 누수로 이어진다.
```cs
public class GenericPool<T> where T : new()
{
    private readonly Stack<T> _pool = new Stack<T>();

    public void Clear()
    {
        while (_pool.Count > 0)
        {
            var item = _pool.Pop();
            // T가 IDisposable을 구현했을 수도 있으므로 확인 후 해제
            if (item is IDisposable disposable)
            {
                disposable.Dispose();
            }
        }
    }
}
```

### IDisposable - "나는 명시적으로 정리해야 할 리소스를 갖고 있다"
- 파일 핸들, 소켓, 네이티브 메모리처럼 GC가 자동으로 못 치워주는 리소스를 들고 있는 타입이 구현하는 인터페이스.

```cs
public interface IDisposable
{
    void Dispose();
}
```
- 이게 전부다.
- 객체를 다 썼다면 반드시 `Dispose()`를 호출해서 리소스를 정리해달라는 계약을 표현하는 것이다.

```cs
public class FileLogger : IDisposable
{
    private StreamWriter _writer = new StreamWriter("log.txt");

    public void Write(string msg) => _writer.WriteLine(msg);

    public void Dispose()
    {
        _writer.Dispose(); // 파일 핸들 해제
    }
}

// using 구문을 쓰면 블록이 끝날 때 Dispose()가 자동 호출됨 (예외가 나도 보장됨)
using (var logger = new FileLogger())
{
    logger.Write("Hello");
} // 여기서 자동으로 logger.Dispose() 호출

// C# 8 이후 문법 (using 선언)
using var logger2 = new FileLogger();
logger2.Write("World");
// 현재 스코프(메서드) 끝날 때 자동 Dispose
```

## 공변성과 반공변성을 지원하라
- 제네릭 인터페이스에 `out`(공변)/`in`(반공변) 키워드를 붙이면 `IEnumerable<Derived>`를 `IEnumerable<Base>`로 다루는 등 더 유연한 다형성이 가능해진다.

- 유니티
	- 일반적인 경우 직접 만들 일은 드물고, `IEnumerable<T>`(공변) 등에 이미 있는 BCL 인터페이스를 통해 자연스럽게 혜택을 받는 경우가 대부분이다. 

```cs
IEnumerable<Enemy> enemies = new List<Enemy>();
IEnumerable<Character> characters = enemies; // 공변성 덕분에 컴파일 가능 (Enemy : Character라면)
```

### 공변성, 반공변성
- **타입 `B`가 `A`의 파생 타입일 때, `제네릭<B>`를 `제네릭<A>`로 취급해도 되는가?** 에 대한 규칙.
- C#의 제네릭은 기본적으로 이게 되지 않는다. 
- 예를 들어 `Enemy: Character`라고 해도, `List<Enemy>`는 `List<Character>`로 취급될 수 없다. 
- 왜냐하면..
```cs
List<Character> characters = enemies; 
characters.Add(new Player()); // Player도 Character의 파생 타입이라 추가 가능
// 근데 실제로는 enemies라는 List<Enemy>에 Player가 들어가버림 -> 타입 오염!
```

- 그렇기에 `List<T>` 처럼 **값을 넣을 수도 뺄 수도 있는 타입은 공변/반공변을 지원하지 않는다.**
- 하지만 **읽기 전용, 쓰기 전용으로만 동작이 제한된 인터페이스라면 안전하게 지원할 수 있다.** 이 때 쓰는 게 `out`, `in` 키워드이다.

> 아래 예제는 `Enemy: Character` 관계임.

- 공변성`out` - 꺼내기(=읽기)만 한다면 안전
```cs
public interface IEnumerable<out T> // 이미 BCL에 이렇게 정의되어 있음
{
    IEnumerator<T> GetEnumerator();
    // T를 매개변수로 "받는" 메서드가 없음 -> 오직 T를 "내보내기"만 함 -> 안전
}

IEnumerable<Enemy> enemies = new List<Enemy> { new Enemy(), new Enemy() };
IEnumerable<Character> characters = enemies; // 공변성 덕분에 OK (IEnumerable<T>는 out T)

foreach (Character c in characters) // Enemy들을 Character로서 읽기만 함 -> 안전
{
    Debug.Log(c.Name);
}
```

- 반공변성`in` - 집어넣기(=쓰기)만 한다면 안전.
```cs
public interface IComparer<in T> // BCL 정의
{
    int Compare(T x, T y);
    // T는 오직 매개변수(입력)로만 쓰임 -> 반환값으로 나오지 않음 -> 안전
}

public class ByNameComparer : IComparer<Character> // Character 기준으로 만든 비교자
{
    public int Compare(Character x, Character y) => string.Compare(x.Name, y.Name);
}

List<Enemy> enemies = new List<Enemy>();
IComparer<Enemy> comparer = new ByNameComparer(); // 반공변성 덕분에 OK (IComparer<T>는 in T)
enemies.Sort(comparer.Compare); // Character용 비교자를 Enemy 정렬에 그대로 재사용
```

>[!warning]
>- 여기서의 `in, out`과 다른 곳에서 쓰이는 `in, out`은 용어만 공유하는 다른 개념이다. 
>	- 예를 들어 `TryGetComponent`에 들어가는 `out`은 공변성이랑은 관계 없는, 반환값 외의 내부 변수를 밖으로 끄집어내서 쓰겠다는 의미임. 

### BCL(Base Class Library)
- `.NET`이 기본으로 제공하는 표준 라이브러리.
- `System, System.Collections.Generic, List<T>, Dictionary<TKey, TValue>, string, DateTime` 등등 **C# 쓸 때 사용하는 대부분의 것들이 전부 BCL에 속한다.** 

- 유니티와의 관계
- 유니티는 BCL 위에 자신의 라이브러리 `UnityEngine 네임스페이스`를 얹어서 쓰는 구조다.
```
UnityEngine (유니티가 만든 게임 엔진 전용 라이브러리)
        ↑ 이 위에 얹어져 있음
BCL / .NET 표준 라이브러리 (List<T>, string, IDisposable 등 — 유니티가 만든 게 아님)
        ↑ 이 위에서 실행됨
C# 언어 자체 (문법: var, class, if 등)
```

- 구분 예시
	- C# 언어 문법 : `var, class, is, as` 
	- BCL : `List<T>, Dictionary<T>, IDisposable, IComparable` 
	- 유니티 : `GameObject, Transform, MonoBehaviour, Destroy()` 

## 타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 이용하라
- C#에서는 특정 메서드를 가져야 한다는 제약을 제네릭 `where`으로 걸 수 없다.
- 대신 `Func<T, TResult>` 같은 델리게이트를 매개변수로 받아 로직을 주입하자.

- 유니티
	- 정렬, 필터링 유틸리티를 직접 만들 때 자주 사용한다. 
	- [[Effective Csharp - 1. Csharp 언어 요소#델리게이트를 이용해 콜백을 표현하라]]와 같은 맥락.
```cs
// "T가 비교 가능해야 한다"는 제약을 걸 수 없으므로, 비교 로직 자체를 주입받음
public static T FindMax<T>(IEnumerable<T> items, Func<T, float> selector)
{
    T best = default;
    float bestValue = float.MinValue;
    foreach (var item in items)
    {
        float value = selector(item);
        if (value > bestValue) { bestValue = value; best = item; }
    }
    return best;
}

// 사용: 체력이 가장 높은 적 찾기
var strongest = FindMax(enemies, e => e.Hp);
```

## 베이스 클래스나 인터페이스에 대해 제네릭을 특화하지 말라
- 오버로딩 시 제네릭과 일반 메서드가 섞여 있으면 컴파일러의 `메서드 해상도Method Resolution` 규칙 때문에 원치 않는 메서드가 호출될 수 있다.

- 유니티
	- 자주 마주치는 상황은 아니다. 커스텀 유틸리티 라이브러리를 만들 때 오버로드를 헷갈리게 설계하지 않는 것만 주의하도록 설계하면 됨.

```cs
public class Logger
{
    public void Log<T>(T value) => Debug.Log($"제네릭: {value}");
    public void Log(Enemy value) => Debug.Log($"특화: {value.Name}"); // 헷갈림의 원인

    // Log(enemy) 호출 시 어느 게 불릴지 직관과 다를 수 있음 -> 이런 조합 자체를 피하는 게 상책
}
```

## 타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제네릭 메서드를 정의하라
- 클래스 전체를 제네릭으로 만들기보다, 메서드만 제네릭으로 만들면 호출부에서 타입 인자를 생략해도 컴파일러가 추론해줘서 훨씬 편하다.

- 유니티
	- 실전에서 자주 쓰는 항목이다.
	- `List<T>` 셔플이나 배열 유틸 등 **상태를 갖지 않는 헬퍼는 항상 이 형태로 만드는 걸 권장**한다.

```cs
// 나쁜 예: 클래스 자체를 제네릭으로 (상태가 없는데도 타입 인자를 매번 명시해야 함)
public class ListShuffler<T>
{
    public void Shuffle(List<T> list) { /* ... */ }
}
var shuffler = new ListShuffler<Enemy>(); // 불필요한 인스턴스화
shuffler.Shuffle(enemyList);


// 좋은 예: 메서드만 제네릭 (타입 추론 덕분에 호출이 간결)
public static class ListUtils
{
    public static void Shuffle<T>(this List<T> list)
    {
        for (int i = list.Count - 1; i > 0; i--)
        {
            int j = UnityEngine.Random.Range(0, i + 1);
            (list[i], list[j]) = (list[j], list[i]);
        }
    }
}
enemyList.Shuffle(); // <Enemy> 명시 안 해도 됨
```


## 제네릭 인터페이스와 논제네릭 인터페이스를 함께 구현하라
- `IEnumerable<T>`와 `IEnumerable`을 함께 구현하는 식으로, 하위 호환성이나 비제네릭 .NET 라이브러리 연동을 위해 두 버전을 같이 지원하라.

- 유니티
	- 거의 마주칠 일은 없다. 
	- `foreach`를 지원하는 나만의 컬렉션 타입을 만들 때 정도만 기억하면 된다.

## 인터페이스는 간략히 정의하고 확장 메서드로 기능을 더하라 
- 인터페이스에 수많은 메서드를 모두 넣으면 구현하기가 힘들어진다. 
- 핵심만 정의하고, 유틸리티는 확장 메서드로 정의하는 것이 좋다. 

- 유니티
	- **확장 메서드는 정말 애용된다.**
	- `Transform`, `GameObject`, `Component` 등의 **유니티 기본 타입에 기능을 덧붙일 때 유용**하다. 상속이 불가능한 타입들이라 확장 메서드가 유일한 확장 수단이기도 하다.

```cs
public interface IDamageable
{
    void TakeDamage(int amount); // 핵심 멤버만
}

// 부가 기능은 확장 메서드로
public static class DamageableExtensions
{
    public static void Heal(this IDamageable target, int amount)
        => target.TakeDamage(-amount); // TakeDamage 하나만으로 파생 기능 제공

    public static bool IsDead(this IDamageable target, int currentHp)
        => currentHp <= 0;
}

// 확장 메서드는 대충 이런 식으로 사용함
target.Heal(amount);
target.IsDead(currentHp); 
```
> **인터페이스 + 확장 메서드로 기능을 구현**하고 있다. 상상도 못한..!


## 확장 메서드를 이용해 구체화된 제네릭 타입을 개선하라
- 특정 제네릭 타입에만 동작하는 로직이 필요할 때, 확장 메서드를 쓰면 상속 없이도 쉽게 추가할 수 있다. 

- 유니티
	- `List<GameObject>`, `List<Transform>` 등 자주 쓰는 컬렉션에 유니티 특화 헬퍼를 붙이는 패턴으로 실무에서 매우 자주 쓰인다.

```cs
public static class TransformListExtensions
{
    // List<Transform>에만 동작하는 특화 로직 (List<T> 전체가 아니라 Transform으로 구체화)
    public static Transform FindClosest(this List<Transform> transforms, Vector3 origin)
    {
        Transform closest = null;
        float minDist = float.MaxValue;
        foreach (var t in transforms)
        {
            float d = Vector3.Distance(origin, t.position);
            if (d < minDist) { minDist = d; closest = t; }
        }
        return closest;
    }
}

// 사용
var nearest = spawnPoints.FindClosest(player.position);
```