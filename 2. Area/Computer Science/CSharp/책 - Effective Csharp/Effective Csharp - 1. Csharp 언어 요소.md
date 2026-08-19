#Csharp #EffectiveCsharp 

1. [[#1. 지역변수를 선언할 때는 `var`을 쓰는 게 낫다|1. 지역변수를 선언할 때는 `var`을 쓰는 게 낫다]]
2. [[#2. `const`보다는 `readonly`가 좋다|2. `const`보다는 `readonly`가 좋다]]
3. [[#3. 캐스팅보다는 `is, as`가 좋다|3. 캐스팅보다는 `is, as`가 좋다]]
4. [[#4. `string.Format()` 대신 보간 문자열 `$"{var}"`을 사용하라|4. `string.Format()` 대신 보간 문자열 `$"{var}"`을 사용하라]]
5. [[#5. 문화권 별로 다른 문자열을 생성하려면 FormattableString을 사용하라|5. 문화권 별로 다른 문자열을 생성하려면 FormattableString을 사용하라]]
6. [[#6. `nameof()` 연산자를 적극 활용하라|6. `nameof()` 연산자를 적극 활용하라]]
7. [[#7. 델리게이트를 이용해 콜백을 표현하라|7. 델리게이트를 이용해 콜백을 표현하라]]
8. [[#8. 이벤트 호출 시에는 `null` 조건 연산자를 사용하라|8. 이벤트 호출 시에는 `null` 조건 연산자를 사용하라]]
9. [[#9. 박싱과 언박싱을 최소화하라|9. 박싱과 언박싱을 최소화하라]]
10. [[#10. 베이스 클래스가 업그레이드된 경우에만 `new` 한정자를 사용하라|10. 베이스 클래스가 업그레이드된 경우에만 `new` 한정자를 사용하라]]


- 각 목차와 세부적인 내용을 Gemini를 통해 정리했다. 유니티에 어울리는 조언과 코드를 Claude로부터 받아서 정리한 내용. 
- 복붙하면 머리에 안 들어와서 직접 타이핑해서 정리함.

## 지역변수를 선언할 때는 `var`을 쓰는 게 낫다
- 컴파일러가 정확한 타입을 추론하게 함으로써, 코드 가독성을 높이고 의도치 않은 암시적 형변환 오류를 방지할 수 있다. 

- 유니티 관련
	- `GetComponent<T>`, `Instantiate<T>` 등 **타입이 코드에 이미 드러나는 경우는 `var`을 쓰면 더 명확해진다.** 
	- 반대로 **반환 타입을 한눈에 알 수 없는 경우는 `var` 사용을 피하고 명시적 타입을 쓰는 게 협업 시에 더 낫다.**

```cs
// 타입이 오른쪽에 이미 드러남 -> var 권장
var rb = GetComponent<Rigidbody>();
var enemies = new List<Enemy>();

// 반환 타입이 불명확 -> 명시적 타입 권장 
PlayerState state = _stateMachine.GetCurrentState();
```


## `const`보다는 `readonly`가 좋다
- `const`는 컴파일 타임 상수로 어셈블리 간의 버전을 맞추기 까다롭다.
	- 컴파일 타임에 값이 박혀버리는 것을 `inline`이라고 한다. 값이 바뀔 경우, 참조하는 어셈블리를 모두 재컴파일해야 한다. 
- `readonly`는 런타임 상수로 유연성과 유지보수성이 훨씬 뛰어나다.

- 유니티 관련
	- **유니티는 `const`, `readonly` 필드를 모두 인스펙터에 노출할 수 없다.** 
	- **기획자가 인스펙터에서 조정할 필요가 있다면 `[SerializeField] private float`를 쓰고, 코드 내부에서 쓰는 진짜 상수일 때만 `readonly`** 를 쓴다. 
	- 나중에 바뀔 가능성이 있는 값은 `const`로 잡지 않는 것을 권장.

```cs
public class WeaponConfig : MonoBehaviour
{
    // 절대 안 바뀌는 진짜 상수
    public const int MaxAmmoTypes = 5;

    // 런타임에 초기화되거나, 어셈블리 독립성이 필요한 값
    public static readonly float BaseDamage = LoadBalanceValue();

    // 인스펙터 노출이 필요하면 SerializeField (const/readonly 불가)
    [SerializeField] private float _fireRate = 0.2f;

    static float LoadBalanceValue() => 10f;
}
```


## 캐스팅보다는 `is, as`가 좋다
- 예외를 발생시키는 강제 캐스팅보다 `as`를 쓰면 실패 시 `null`을 반환하므로 더 안전하고 성능 상 유리하다. 

- 유니티 관련
	- `(T)obj` 강제 캐스팅은 실패 시 예외를 던지나, `as`는 실패 시 `null`을 반환한다.
	- 충돌 처리에서 상대방 컴포넌트를 가져올 때 자주 쓰이는 패턴이며, **`C# 7` 이후 패턴 매칭`is T variable`을 쓰면 캐스팅과 null 체크를 한 줄로 끝낼 수 있다.**

```cs
void OnTriggerEnter(Collider other)
{
	// 강제 캐스팅(비권장) - Enemy가 아니라면 InvalidCastException
	Enemy enemy = (Enemy)other.GetComponent<Character>();
	
	// as 사용
	var enemy = other.GetComponent<Enemy>();
	if (enemy != null)
	{
		enemy.TakeDamage(10);
	}
	
	// 패턴 매칭 - 가장 권장
	if (other.TryGetComponent<Enemy>(out var enemyPm))
	{
		enemyPm.TakeDamage(10);
	}
}
```
> 유니티의 경우 `GetComponent<T>()`가 실패 시 `null`을 반환하므로, `TryGetComponent<T>()` 내지는 `null` 체크와 함께 쓰는 게 사실상 표준이다.


## `string.Format()` 대신 보간 문자열 `$"{var}"`을 사용하라
- `보간 문자열Interpolated String`은 가독성이 높고 컴파일러가 인자의 개수나 타입을 체크해 실수를 줄여준다.
	- 이건 애초에 **`string.Format()`을 쓴 적이 아예 없다**고 봐야 해서 자세히 정리하진 않겠음

- 유니티 관련
	- **`UI` 텍스트 갱신 시, `TMP_Text.text` 등에 매 프레임 문자열 보간을 쓰면 매번 새 문자열 힙 할당이 발생해 GC 부담이 커진다.** 
	- **자주 갱신되는 UI**는 보간 문자열보다는 **`StringBuilder`나 값이 바뀔 때만 갱신하는 방식을 고려**할 수 있다. 

```cs
// 좋은 예: 가독성 (드물게 갱신되는 텍스트)
_resultText.text = $"점수: {score}, 남은 시간: {timeLeft:F1}초";

// 매 프레임 갱신되는 HUD라면 캐싱/조건부 갱신 고려
void UpdateHpText(int hp)
{
    if (hp == _lastHp) return; // 값이 바뀔 때만 문자열 생성
    _lastHp = hp;
    _hpText.text = $"HP: {hp}";
}
```


## 문화권 별로 다른 문자열을 생성하려면 FormattableString을 사용하라
- 특정 문화권에 종속된 날짜/숫자 포맷이 필요할 때, 문자열 보간 기능과 함께 사용해서 안전하게 포맷팅할 수 있다. 

- 유니티
	- 글로벌 서비스의 경우 가격, 날짜, 숫자 표기가 지역마다 다르다.
	- **실무에선 유니티의 로컬라이제이션 패키지를 함께 쓰는 경우가 더 많다.** 

```cs
using System;
using System.Globalization;

FormattableString message = $"가격: {price:C}";

string korean = message.ToString(new CultureInfo("ko-KR")); // 가격: ₩1,000
string english = message.ToString(new CultureInfo("en-US")); // 가격: $1,000.00

Debug.Log(korean);
```


## `nameof()` 연산자를 적극 활용하라
- 변수, 프로퍼티의 이름을 하드코딩된 문자열 대신 `nameof()`로 가져오면 이름 변경(리팩터링) 시 오류를 컴파일 타임에 방지할 수 있다.

- 유니티
	- 인스펙터 필드 이름을 참조하는 커스텀 에디터나 `SerializedProperty` 작성 시 특히 유용하다. 필드명을 바꿨는데 문자열은 그대로인 실수를 방지한다. 

```cs
public class Player : MonoBehaviour
{
    [SerializeField] private float _moveSpeed;

    void SetSpeed(float speed)
    {
        if (speed < 0)
            throw new ArgumentOutOfRangeException(nameof(speed), "속도는 음수가 될 수 없습니다.");
        _moveSpeed = speed;
    }
}

// 커스텀 에디터에서
[CustomEditor(typeof(Player))]
public class PlayerEditor : Editor
{
    void OnEnable()
    {
        // "_moveSpeed" 문자열 대신 nameof 사용 권장 (private 필드라 직접은 불가하지만
        // public/internal 필드나 프로퍼티라면 nameof(Player.MoveSpeed) 형태로 활용)
        serializedObject.FindProperty("_moveSpeed");
    }
}
```

> [!question]
> 이거 정확한 의미가 헷갈린다. `_moveSpeed`를 `MoveSpeed`라는 프로퍼티로 노출시키면 `"MoveSpeed"`로 쓰지 말고 `nameof()`을 써서 넣으라는 얘기인가?

- 예제를 들어보자. `MoveSpeed`라는 프로퍼티가 있다고 가정한다.
```cs
// 나쁜 예: 문자열 하드코딩
throw new ArgumentException("MoveSpeed는 음수가 될 수 없습니다.");

// 좋은 예: nameof 사용
throw new ArgumentException($"{nameof(MoveSpeed)}는 음수가 될 수 없습니다.");
```

만약 변수명 `MoveSpeed`를 `Speed`로 바꾼다면
- `MoveSpeed`는 프로퍼티의 이름이 바뀌더라도 아무런 에러를 발생시키지 않는다. 문자열을 하드코딩했기 때문이다. 컴파일도 되고 동작은 하지만 실제로 변수 이름은 없는 런타임 버그가 된다.
- `nameOf(MoveSpeed)`는 실제 심볼을 참조하므로, `MoveSpeed`가 없어진 순간 컴파일 에러가 나서 바로 고쳐야 한다. 

즉, 핵심은 **`nameof()`을 쓰면 오류를 바로 알아챌 수 있다**는 것. 나중에 귀찮아질 수 있는 이슈를 방지한다. 



## 델리게이트를 이용해 콜백을 표현하라
- C#에서 콜백이 필요할 때는 인터페이스를 새로 정의하지 않고, 이미 최적화된 델리게이트(`Func`, `Action` 등)를 사용하는 게 더 유연하다.

- 유니티
	- 비동기 작업 완료 콜백을 넘길 때 매우 흔하게 쓰인다.
	- `UnityEvent`도 인스펙터에 노출 가능한 델리게이트 대안이나, 순수 코드 레벨의 콜백이라면 `Action / Func`이 훨씬 가볍고 빠르다.

```cs
public class ResourceLoader : MonoBehaviour
{
    public void LoadAsync(string key, Action<GameObject> onComplete)
    {
        StartCoroutine(LoadRoutine(key, onComplete));
    }

    IEnumerator LoadRoutine(string key, Action<GameObject> onComplete)
    {
        yield return new WaitForSeconds(1f); // 실제로는 Addressables 로딩 등
        var obj = new GameObject(key);
        onComplete?.Invoke(obj);
    }
}

// 사용
loader.LoadAsync("Enemy_01", enemyObj =>
{
    enemyObj.transform.position = spawnPoint.position;
});
```


## 이벤트 호출 시에는 `null` 조건 연산자를 사용하라
- `Event?.Invoke()`를 사용하면 멀티스레드 환경에서도 델리게이트가 중간에 해제되어 발생하는 `NullReferenceException`을 안전하게 막을 수 있다.

- 유니티
	- **유니티의 기본 옵션은 메인 스레드 단일 실행이 기본**이라서 멀티스레드 레이스 컨디션은 드물다.
	- 그걸 감안해도 **`?.Invoke()`라는 패턴은 구독자가 아예 없는 경우를 처리할 때도 유용해서 습관적으로 쓰는 걸 권장**한다.
	- 오브젝트 풀링 시스템에서 오브젝트가 반환되며 이벤트 구독이 해제되는 타이밍 이슈를 예방하는 데에도 도움이 된다.

```cs
public class Health : MonoBehaviour
{
    public event Action<int> OnDamaged;
    public event Action OnDied;

    private int _current = 100;

    public void TakeDamage(int amount)
    {
        _current -= amount;

        // 구독자가 없거나, 멀티스레드 상황에서 안전하게 호출
        OnDamaged?.Invoke(amount);

        if (_current <= 0)
        {
            OnDied?.Invoke();
        }
    }
}
```


## 박싱과 언박싱을 최소화하라
- 값 타입을 참조 타입으로 변환하는 박싱 연산은 성능 저하와 가비지 컬렉션 부하를 유발한다. 제네릭을 사용해 방지해야 한다. 

- 유니티
	- **실전적으로 가장 중요한 항목** 중 하나. 
	- `Update()`에서의 박싱은 GC Spike와 프레임 드랍으로 직결된다.
	- **`ArrayList, Hashtable` 같은 비제네릭 컬렉션을 절대 쓰지 말 것.**
	- **`List<T>`, `Dictionary<TKey, TValue>` 사용을 권장한다.**
	- `Animator.SetFloat`, `SetBool` 등은 이미 오버로드가 값 타입 전용이라 안전하지만, 직접 만든 이벤트 시스템에서 `object` 타입 매개변수를 쓰면 박싱이 자주 발생한다.

```cs
// 나쁜 예: 박싱 발생
void LogValue(object value) // int, float이 들어오면 박싱됨
{
    Debug.Log(value);
}
LogValue(42); // int -> object 박싱

// 좋은 예: 제네릭으로 박싱 회피
void LogValue<T>(T value)
{
    Debug.Log(value);
}
LogValue(42); // 박싱 없음

// 유니티 매 프레임 코드에서 특히 주의
void Update()
{
    // 나쁜 예: ArrayList에 값 타입 추가 시 매번 박싱
    // _arrayList.Add(transform.position.x);

    // 좋은 예: 제네릭 컬렉션
    _floatList.Add(transform.position.x); // List<float>, 박싱 없음
}
```

- 박싱, 언박싱이 기억나지 않는다? :  [[CS - 박싱과 언박싱]] 참조.

>[!question] 
>- 비제네릭 컬렉션과 제네릭 컬렉션은 뭐가 다른 거임?

- **비제네릭 컬렉션은 내부적으로 모든 요소를 `object` 타입으로 저장**한다.
- **제네릭 컬렉션은 저장할 타입`T`를 실제로 알고 있어서 원래 타입 그대로 저장**한다. 

실제로는 비제네릭 컬렉션을 쓸 일이 거의 없다.

## 베이스 클래스가 업그레이드된 경우에만 `new` 한정자를 사용하라
- `new` 한정자로 메서드를 숨기는 것은 다형성을 파괴해 개발자에게 혼동을 준다. 불가피한 하위 호환성 유지 시에만 제한적으로 써야 한다. 

- 유니티
	- **`MonoBehaviour` 상속 구조의 `Start()`, `Update()` 등**은 애초에 `virtual`이 아니라서 `new`로 가릴 필요가 없다. **각 클래스에서 독립적으로 정의되는 구조**이다. 
	- 단, 실제로 문제가 되는 경우는 **직접 만든 상속 계층**으로, 자식 계층이 부모 타입 변수로 다뤄질 때 `new`로 숨긴 메서드가 호출되지 않고 부모 메서드가 호출되는 버그가 매우 흔하다.

```cs
public class Enemy : MonoBehaviour
{
    public virtual void Attack()
    {
        Debug.Log("일반 공격");
    }
}

public class BossEnemy : Enemy
{
    // 나쁜 예: new로 숨기면 다형성이 깨짐
    // public new void Attack() => Debug.Log("보스 공격");

    // 좋은 예: override로 진짜 재정의
    public override void Attack()
    {
        Debug.Log("보스 공격 (범위 공격)");
    }
}

// 사용
Enemy e = new BossEnemy();
e.Attack();
// new 사용 시: "일반 공격" 출력 (의도와 다름, 버그의 온상)
// override 사용 시: "보스 공격 (범위 공격)" 출력 (의도대로 동작)
```

