#Csharp #EffectiveCsharp

1. [[#컬렉션을 반환하기보다 `Iterator`을 반환하는 게 더 낫다|컬렉션을 반환하기보다 `Iterator`을 반환하는 게 더 낫다]]
2. [[#(유니티에서는 X) 루프보다 쿼리 구문이 낫다|(유니티에서는 X) 루프보다 쿼리 구문이 낫다]]
3. [[#쿼리 구문과 메서드 구문을 적절히 혼용하라|쿼리 구문과 메서드 구문을 적절히 혼용하라]]
4. [[#Action, Predicate, Func과 델리게이트를 분리하라|Action, Predicate, Func과 델리게이트를 분리하라]]
5. [[#필요한 시점에 필요한 요소를 생성하라(지연 생성)|필요한 시점에 필요한 요소를 생성하라(지연 생성)]]
6. [[#함수를 매개변수로 사용해 결합도를 낮추라|함수를 매개변수로 사용해 결합도를 낮추라]]
7. [[#확장 메서드는 절대로 오버로딩하지 말라|확장 메서드는 절대로 오버로딩하지 말라]]
8. [[#쿼리 표현식과 메서드 호출 구문이 어떻게 대응되는지 이해하라|쿼리 표현식과 메서드 호출 구문이 어떻게 대응되는지 이해하라]]
9. [[#쿼리를 사용할 때는 즉시 평가보다 지연 평가가 낫다|쿼리를 사용할 때는 즉시 평가보다 지연 평가가 낫다]]
10. [[#메서드보다 람다 표현식이 낫다|메서드보다 람다 표현식이 낫다]]
11. [[#델리게이트나 람다 표현식 안에서 예외를 던지지 말라|델리게이트나 람다 표현식 안에서 예외를 던지지 말라]]
12. [[#지연 수행과 즉시 수행을 구분하라|지연 수행과 즉시 수행을 구분하라]]
13. [[#값비싼 리소스를 캡처하지 말라|값비싼 리소스를 캡처하지 말라]]
14. [[#`IEnumerable` 데이터 소스와 `IQueryable` 데이터 소스를 구분하라|`IEnumerable` 데이터 소스와 `IQueryable` 데이터 소스를 구분하라]]
15. [[#쿼리 결과의 의미를 명확히 하라|쿼리 결과의 의미를 명확히 하라]]
16. [[#바인딩된 변수는 수정하지 말라|바인딩된 변수는 수정하지 말라]]


---
>[!note]
>- 유니티 관점에서 볼 점
>- 이 장의 항목은 "이터레이터 `yield return`과 지연 평가`deferred execution`, 클로저`closure`의 동작 원리를 이해하라"는 한 가지 주제로 수렴한다. 이는 **유니티에서 GC 할당을 몰래 유발하는 주범**이기도 하다.
>- LINQ를 어떻게 잘 쓰는가? 보다는, **LINQ를 매 프레임 실행되는 코드에 쓰지 않는 게 나은 이유**를 이해하는 게 더 도움이 된다. 

## 컬렉션을 반환하기보다 `Iterator`을 반환하는 게 더 낫다
- `List<T>`를 통째로 만들어 반환하는 것보다, `yield return`으로 하나씩 생성해서 반환하면 호출자가 필요한 만큼만 순회할 수 있고 불필요한 전체 컬렉션 할당을 피할 수 있다.

- 유니티
	- `yield return` 키워드는 코루틴에서 이미 익숙한 개념이다. 코루틴 또한 이터레이터 패턴을 활용한 문법으로, 원리를 이해하면 코루틴 동작도 더 명확해진다.

```cs
// 나쁜 예: 전체 결과를 리스트로 만들어 반환 (한 번에 다 계산 + 할당)
public List<Enemy> GetEnemiesInRange(float range)
{
    var result = new List<Enemy>();
    foreach (var e in _allEnemies)
    {
        if (Vector3.Distance(e.transform.position, transform.position) <= range)
            result.Add(e);
    }
    return result;
}

// 좋은 예 : 이터레이터로 필요한 만큼만 지연 생성
public IEnumerable<Enemy> GetEnemiesInRangeLazy(float range)
{
    foreach (var e in _allEnemies)
    {
        if (Vector3.Distance(e.transform.position, transform.position) <= range)
            yield return e; // 호출부가 실제로 요청할 때만 계산
    }
}

// 1번째 결과만 필요하다면, 지연 버전은 전체를 다 돌지 않고 멈출 수 있다.
var first = GetEnemiesInRangeLazy(10f).FirstOrDefault();
```

> 이거는 내 프로젝트에 쓸 수 있는 요소로 보인다. `Enemy`나 `Operator`의 공격 대상을 감지하는 로직.


## (유니티에서는 X) 루프보다 쿼리 구문이 낫다
- `foreach` 루프를 직접 짜는 것보다 `Where, Select` 같은 `LINQ` 쿼리가 가독성이 좋고 의도가 명확하다.

- 유니티
	- 하지만 **유니티의 매 프레임 도는 코드에서는 정반대로 `LINQ`를 피하고 일반 `for / foreach` 루프를 쓰는 걸 강력히 권장**한다. 
	- **LINQ 메서드는 내부적으로 이터레이터 객체, 클로저 등을 힙에 할당**하기 때문에, 매 프레임 호출 시 GC 부하가 누적되는 이슈가 있기 때문이다.

```cs
// 가독성은 좋지만, Update()에서 매 프레임 호출하면 GC 할당 발생
void Update()
{
    var closeEnemies = _enemies.Where(e => Vector3.Distance(e.position, transform.position) < 5f).ToList();
}

// 유니티 핫패스에서는 이렇게 (할당 없음)
void Update()
{
    for (int i = 0; i < _enemies.Count; i++)
    {
        if (Vector3.Distance(_enemies[i].position, transform.position) < 5f)
        {
            // 처리
        }
    }
}
```

물론 일회성으로 사용되는 코드는 상관 없다. `Update()`류에서 주의하라는 것.


## 쿼리 구문과 메서드 구문을 적절히 혼용하라
- 메서드가 `IEnumerable<T>`를 받아서 `IEnumerable<T>`를 반환하도록 설계하면 LINQ 메서드처럼 체이닝해서 쓸 수 있는 유연한 API가 된다.

- 유니티
	- 인벤토리 필터링, 스킬 목록 처리 등 여러 단계를 거치는 데이터 가공 로직을 유틸리티로 뽑을 때 적용할 수 있는 설계 원칙 정도로 참고하면 된다. 실전에서 손수 만들 일이 많지 않다. 
```cs
public static IEnumerable<Item> OnlyEquippable(this IEnumerable<Item> items)
    => items.Where(i => i.IsEquippable);

public static IEnumerable<Item> SortByRarity(this IEnumerable<Item> items)
    => items.OrderByDescending(i => i.Rarity);

// 체이닝 가능
var result = inventory.OnlyEquippable().SortByRarity();
```

## Action, Predicate, Func과 델리게이트를 분리하라
- 무엇을 할지(로직)와 어떻게 순회할지(반복 방식)을 하나의 메서드에 섞지 말고, 델리게이트로 분리해서 재사용성을 높여라.

- 유니티
	- [[Effective Csharp - 1. Csharp 언어 요소#델리게이트를 이용해 콜백을 표현하라|델리게이트를 이용해 콜백을 표현하라]] 및 [[Effective Csharp - 3. 제네릭 활용#타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 이용하라|타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 이용하라]] 와 비슷한 이야기. 
	- 예시로, 모든 자식 오브젝트에 대해 어떤 동작을 수행하는 유틸리티를 만들 때 순회 동작과 실제 동작을 분리하면 여러 상황에 재사용할 수 있다.

```cs
// 순회 로직만 담당 (재사용 가능)
public static void ForEachChild(this Transform parent, Action<Transform> action)
{
    for (int i = 0; i < parent.childCount; i++)
        action(parent.GetChild(i));
}

// 사용: 동작만 바꿔 끼움
someParent.ForEachChild(child => child.gameObject.SetActive(false));
someParent.ForEachChild(child => Debug.Log(child.name));
```

## 필요한 시점에 필요한 요소를 생성하라(지연 생성)
- 시퀀스의 모든 요소를 미리 만들어두지 말고, 실제로 소비될 때 그 순간에 생성하라.
- [[Effective Csharp - 4. LINQ 활용#컬렉션을 반환하기보다 `Iterator`을 반환하는 게 더 낫다|Iterator을 반환하는 게 더 낫다]]와 같은 맥락.

- 유니티
	- 대량의 데이터(절차적 던전의 방, 아이템 조합 목록 등)를 다룰 때 전부 미리 만들어 메모리에 올리는 대신 필요한 순간에만 생성하면 초기 로딩 시간과 메모리 사용량을 줄일 수 있다.

```cs
// 나쁜 예: 조합 가능한 모든 아이템을 미리 다 계산 (메모리, 시간 낭비)
public List<ItemCombo> GetAllCombos()
{
    var result = new List<ItemCombo>();
    foreach (var a in _items)
        foreach (var b in _items)
            result.Add(new ItemCombo(a, b)); // 아이템 100개면 만 개를 미리 다 생성
    return result;
}

// 좋은 예: 실제로 순회될 때만 생성
public IEnumerable<ItemCombo> GetAllCombosLazy()
{
    foreach (var a in _items)
        foreach (var b in _items)
            yield return new ItemCombo(a, b); // 필요한 만큼만 생성됨
}
```


## 함수를 매개변수로 사용해 결합도를 낮추라
- 델리게이트를 쓰라는 다른 것들과 같은 이야기.
	- [[Effective Csharp - 1. Csharp 언어 요소#델리게이트를 이용해 콜백을 표현하라|델리게이트를 이용해 콜백을 표현하라]] 
	- [[Effective Csharp - 3. 제네릭 활용#타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 이용하라|타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 이용하라]]
	- [[Effective Csharp - 4. LINQ 활용#Action, Predicate, Func과 델리게이트를 분리하라|Action, Predicate, Func과 델리게이트를 분리하라]]

## 확장 메서드는 절대로 오버로딩하지 말라
- 같은 이름의 확장 메서드를 여러 시그니처로 오버로드하면, 원본 타입에 진짜 같은 이름의 인스턴스 메서드가 추가될 때 어떤 게 호출될지 예측하기 어려워진다.

- 유니티
	- 실무에서 자주 마주치는 문제는 아니나, 확장 메서드 라이브러리를 크게 키울 계획이라면 이름이 겹치지 않게끔 신경쓰는 정도로 충분하다.

## 쿼리 표현식과 메서드 호출 구문이 어떻게 대응되는지 이해하라
- `from x in list where ... select ...` 같은 쿼리 구문은 결국 `list.Where(...).Select(...)` 같은 메서드 체인으로 컴파일러가 변환해주는 것 뿐이다.
- 유니티
	- 어차피 대부분 메서드 체인 쓰잖음?

## 쿼리를 사용할 때는 즉시 평가보다 지연 평가가 낫다
- **LINQ 쿼리는 `.ToList()`나 `foreach` 등으로 실제로 순회되기 전까지는 실행되지 않는다.** 이 특성을 이해하고 활용하라.

- 유니티
	- 매우 흔한 실수의 원인이다. 
	- 지연 평가 때문에, 같은 쿼리를 2번 순회하면 계산도 2번 일어난다. 
	- `GameObject`나 `Transform` 같은 유니티 오브젝트를 참조하는 지연 평가된 쿼리가 있는 상황에서, 그 쿼리를 실제로 순회하기 전에 해당 오브젝트가 `Destroy()`되면 이미 파괴된 오브젝트에 접근하는 버그가 생길 수 있다. 
	- **쿼리를 만든 시점과 실행 시점이 다르다**는 것에 유의할 것. 

```cs
var query = _enemies.Where(e => IsExpensiveCheck(e)); // 이 시점에 실행되는 건 없다. 

int count = query.Count(); // 전체 순회 1회, IsExpensiveCheck 모두 호출
var list = query.ToList(); // 전체 순회 1회, IsExpensiveCheck 모두 호출

// 해결 방법 : 1번만 평가되도록 즉시 리스트로 확정짓기
var materialized = _enemies.Where(e => IsExpensiveCheck(e)).ToList();
int count2 = materialized.Count; // 이미 계산된 결과를 사용하므로 재평가 없음
var list2 = materialized; // 마찬가지
```

## 메서드보다 람다 표현식이 낫다
- 한 번만 쓰이는 짧은 델리게이트 로직이라면, 별도 메서드를 정의하는 것보다 람다 식으로 인라인으로 작성하는 게 간편하다.

- 유니티
	- `Update()`에서 쓰는 건 주의하자. 매 프레임 람다식을 만드는 것도 클로저 할당이다. 
	- **람다 자체가 변수를 캡처(Closure)하면 그 람다를 표현하기 위한 객체가 힙에 새로 만들어진다.** 

```cs
void Update()
{
    var target = _enemies.FirstOrDefault(e => e.Hp > 0); // e => e.Hp > 0 은 캡처가 없어 괜찮지만
    var nearTarget = _enemies.FirstOrDefault(e => Vector3.Distance(e.position, transform.position) < range);
    // 여기서 range, transform은 외부 변수를 캡처(클로저) -> 매 프레임 새 델리게이트 객체 할당
}

```

## 델리게이트나 람다 표현식 안에서 예외를 던지지 말라
- LINQ에 넘기는 델리게이트(`Where, Select` 등의 람다) 안에서 예외가 발생하면 지연 평가 특성 때문에 스택 추적이 원래 의도와 다른 위치에서 나타나 디버깅이 훨씬 어려워진다.

- 유니티
	- `NullReferenceException`이 `Where` 안의 람다에서 터지면, 콘솔에 찍히는 에러 위치가 실제로 그 쿼리를 정의한 곳이 아니라 순회한 곳으로 나올 수 있어 원인 추적이 헷갈린다. 
	- 람다 내부에서 `null` 체크를 미리 해두는 습관이 필요하다.

```cs
// 위험: e.Owner가 null이면 예외가 이상한 지점에서 발생한 것처럼 보임
var query = _enemies.Where(e => e.Owner.IsAlive);

// 안전: 람다 내부에서 방어적으로 체크
var query2 = _enemies.Where(e => e.Owner != null && e.Owner.IsAlive);
```

## 지연 수행과 즉시 수행을 구분하라
- [[Effective Csharp - 4. LINQ 활용#쿼리를 사용할 때는 즉시 평가보다 지연 평가가 낫다 | 쿼리를 사용할 때는 즉시 평가보다 지연 평가가 낫다]]와 같은 이야기.
- 즉시 평가 :  `.ToList()`, `.ToArray()`, `.Count()`, `.First()`
- 지연 평가 : `.Where()`, `.Select()`


## 값비싼 리소스를 캡처하지 말라
- 람다식(클로저)이 파일 핸들, 네트워크 연결처럼 값비싼, 혹은 수명이 짧은 리소스를 캡처하면 그 리소스의 생명 주기가 클로저의 생명주기에 얽혀 예기치 않게 오래 살아남거나 이미 해제된 리소스에 접근하는 문제가 생긴다.

- 유니티
	- **정말 자주 발생하는 실전 버그와 직결**된다. 
	- 코루틴이나 콜백 람다가 파괴된 `MonoBehaviour`, `Transform`을 캡처하고 있다면 오브젝트가 `Destroy()` 된 후에도 람다는 그 참조를 계속 들고 있어서 예상치 못한 동작이나 fake null 관련 에러가 발생할 수 있다.

```cs
public class Spawner : MonoBehaviour
{
    void SpawnAndTrack()
    {
        var enemy = Instantiate(enemyPrefab);

        // 위험한 예: 람다가 enemy(비싼 리소스가 될 수 있는 유니티 오브젝트)를 캡처
        StartCoroutine(DelayedCheck(() =>
        {
            // 만약 이 콜백이 실행되는 시점에 enemy가 이미 Destroy() 되어 있다면
            // "fake null" 상태의 오브젝트에 접근하게 됨
            Debug.Log(enemy.transform.position);
        }));
    }

    IEnumerator DelayedCheck(Action callback)
    {
        yield return new WaitForSeconds(5f);
        callback?.Invoke(); // 5초 뒤, enemy가 이미 파괴되어 있을 수 있음
    }
}
```

- 대안 
	- 콜백 실행 직전에 대상이 여전히 유효한지 명시적으로 확인하기
	- 필요한 값만 뽑아서 캡처하고 오브젝트 전체를 캡처하지 않기
```cs
var enemy = Instantiate(enemyPrefab);
Vector3 spawnPos = enemy.transform.position; // 필요한 값만 미리 추출

StartCoroutine(DelayedCheck(() =>
{
    Debug.Log(spawnPos); // 오브젝트 전체가 아니라 값만 캡처했으므로 안전
}));
```

## `IEnumerable` 데이터 소스와 `IQueryable` 데이터 소스를 구분하라
- `IEnumerable<T>`는 메모리 상의 컬렉션을 순회한다.
- `IQueryable<T>`는 쿼리를 SQL 등으로 변환해 원격 데이터 소스에 위임한다. 
- 둘을 구분하지 않으면 전체 데이터를 메모리로 끌어온 뒤 필터링하는 비효율이 생길 수 있다.

- 유니티
	- `IQueryable<T>`는 DB, ORM 등에서 함께 쓰이는 개념이라서 유니티에서 볼 일은 거의 없다.
	- 서버 사이드를 다루는 경우만 알아둘 가치가 있는 정도.

## 쿼리 결과의 의미를 명확히 하라
- 정확히 하나만 있어야 하는 결과는 `Single()`, 여러 개 중 첫 번째만 필요할 때는 `First()`를 써서 코드의 의도와 실제 데이터 상태를 명확히 하라.

- 유니티
	- 씬에서 특정 태그를 가진 오브젝트를 찾을 때 `FirstOrDefault()`를 쓰기 쉬운데, **실제로 반드시 하나만 존재하는 경우라면 `Single()`** 을 쓰는 게 데이터 무결성 검증에도 도움이 된다. 
```cs
// "플레이어는 씬에 반드시 1명"이라는 가정이 있다면
var player = allCharacters.SingleOrDefault(c => c.CompareTag("Player"));
// 만약 실수로 플레이어가 2명 존재하는 상태가 되면 여기서 예외로 바로 드러남
// (FirstOrDefault를 썼다면 이 실수를 조용히 숨기고 그냥 첫 번째만 가져와버림)

// "가장 가까운 적 하나만 필요" 같은 경우엔 First가 자연스러움
var nearest = enemies.OrderBy(e => Vector3.Distance(e.position, playerPos)).First();
```

## 바인딩된 변수는 수정하지 말라
- **람다식이 캡처한 외부 변수를 나중에 수정하면, 그 변수를 참조하는 모든 클로저가 수정된 최신 값을 보게 된다.** 

- 유니티
	- 매우 자주 나오는 버그 패턴.
	- **여러 개의 콜백을 반복문 안에서 만들 때 특히 조심해야 한다.**
```cs
List<Action> callbacks = new List<Action>();

for (int i = 0; i < 5; i++)
{
    callbacks.Add(() => Debug.Log(i)); // i라는 "외부 변수 자체"를 캡처
}

foreach (var cb in callbacks) cb.Invoke();
// 예상: 0, 1, 2, 3, 4
// 실제: 5, 5, 5, 5, 5 - 모든 람다가 동일한 i 변수를 참조하고 있기 때문.
```

```cs
// 해결: 루프 안에서 별도의 지역 변수로 복사한 뒤 그걸 캡처
List<Action> callbacks2 = new List<Action>();
for (int i = 0; i < 5; i++)
{
    int captured = i; // 매 반복마다 새로운 변수가 생성됨
    callbacks2.Add(() => Debug.Log(captured));
}
foreach (var cb in callbacks2) cb.Invoke();
```
- **참고**
	- `foreach(var item in collection)`은 C# 5부터 반복마다 새 변수가 자동으로 생성되도록 변경되어서 이 문제가 없다.
	- 전통적인 인덱스 루프 `for (int i; ...)`문에서 문제가 발생한다. 
	- 유니티에서 여러 UI 버튼에 인덱스별로 다른 콜백을 등록하는 코드를 작성할 때 이 버그가 자주 나온다.
```cs
// 실전 예: 버튼 여러 개에 인덱스별 콜백을 등록할 때
for (int i = 0; i < buttons.Length; i++)
{
    int index = i; // 반드시 복사해야 함
    buttons[i].onClick.AddListener(() => SelectSlot(index));
    // int index = i; 를 빼먹으면 모든 버튼이 마지막 인덱스로만 동작하는 버그가 남
}
```

## 클로저
- [[Csharp - 클로저(Closure)]]에도 정리해뒀음.

