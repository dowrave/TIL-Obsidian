
## 메서드의 실패를 알리기 위해 예외를 사용하라
- 메서드가 실패했을 때 `false`, `null`, 특수한 매직 넘버 `-1` 등을 반환해 조용히 알리는 것보다는, 진짜 예외적인 상황이라면 예외를 던져서 실패를 명확히 알려야 한다.
- **하지만!!!! 유니티라면 예외 케이스가 많다.** 
	- `GetComponent<T>()`는 실패해도 예외를 던지지 않고 `null`을 던진다. "특정 컴포넌트가 없다"는 매우 흔하고 정상적인 경우이기 때문이다.
	- `Update()`마다 컴포넌트가 없을 수도 있다는 이유로 예외처리를 해버리면 성능(예외 처리 비용)과 코드 흐름 모두 망가진다.

```cs
// 진짜 "예외적인" 상황 -> 예외를 던지는 게 맞음
public void LoadConfig(string path)
{
    if (!File.Exists(path))
        throw new FileNotFoundException($"설정 파일을 찾을 수 없습니다: {path}");
    // 설정 파일이 없다는 건 정말 예상 밖의 심각한 상황
}

// "있을 수도, 없을 수도 있는" 정상적인 상황 -> null/bool 반환이 맞음
public bool TryGetTargetEnemy(out Enemy enemy)
{
    enemy = _enemies.FirstOrDefault(e => e.IsInRange);
    return enemy != null; // 적이 없는 건 흔한 정상 상황, 예외로 다룰 일이 아님
}
```

- 판단 기준 : 호출하는 쪽의 코드에서 매번 체크해야 할 만큼 **흔한 일이라면 반환값**을, 정말 **예상 밖의 비정상적 상태라면 예외**로 처리하면 됨.

## 리소스 정리를 위해 `using`과 `try/finally`를 이용하라
- `IDisposable`을 구현한 리소스는 `using` 블록으로 감싸서 예외가 발생하더라도 반드시 `Dispose()` 가 호출되도록 보장하라.

- 유니티
	- `NativeArray<T>, NativeList<T>` 등이 `IDisposable`을 구현한 대표적인 요소들로, 이들은 `Dispose()`을 하지 않으면 콘솔에 명시적으로 메모리 누수 경고가 뜬다.
	- 위 예시들은 DOTS/Job System에서 사용함.
```cs
using Unity.Collections;

void ProcessData()
{
    // using으로 감싸면, 아래서 예외가 나더라도 Dispose가 보장됨
    using (var nativeArray = new NativeArray<float>(100, Allocator.Temp))
    {
        for (int i = 0; i < nativeArray.Length; i++)
        {
            nativeArray[i] = ComputeValue(i); // 여기서 예외가 나도 Dispose는 실행됨
        }
    } // 여기서 자동으로 nativeArray.Dispose() 호출

    // 일반 코루틴/비동기 흐름에서 파일 스트림 등을 다룰 때도 동일한 원칙 적용
    using var writer = new StreamWriter("save.json");
    writer.Write(saveData);
}
```
## 사용자 지정 예외 클래스를 완벽히 작성하라
- 커스텀 예외를 만들 땐 표준 생성자 패턴(기본, 메시지, 메시지 + 내부 예외)을 다 구현하고 직렬화 지원까지 고려한 완전한 예외 타입으로 만들어라.

- 유니티
	- 커스텀 예외를 직접 정의하는 일은 `.NET` 서버 개발보다는 적다.
	- 세이브 데이터 로드 실패, 커스텀 에셋 파싱 실패 등 명확한 도메인 에러를 표현하고 싶을 때 유용하다. 
	- 직렬화`ISerializable` 부분은 유니티(특히 `IL2CPP` 빌드)  에서는 신경쓸 필요가 없고, 아래 3가지 생성자만 챙기면 된다.
```cs
public class SaveDataCorruptedException : Exception
{
    public SaveDataCorruptedException() { }
    public SaveDataCorruptedException(string message) : base(message) { }
    public SaveDataCorruptedException(string message, Exception inner) : base(message, inner) { }
}

// 사용
try
{
    var data = JsonUtility.FromJson<SaveData>(json);
    if (data == null)
        throw new SaveDataCorruptedException("세이브 파일 파싱 결과가 비어있습니다.");
}
catch (Exception ex)
{
    throw new SaveDataCorruptedException("세이브 데이터 로딩 실패", ex); // 원본 예외를 inner로 보존
}
```

## 강력한 예외 보증을 준수하는 것이 좋다
- 메서드 실행 중 예외가 발생하더라도, 객체가 일관성 없는 중간 상태로 남아있으면 안 된다. 
	- **전부 성공하거나, 전부 실패한 것처럼 원상복구**되는 둘 중 하나여야 함.

- 유니티
	- 인벤토리 시스템, 세이브 / 로드 처럼 **여러 단계를 거치는 상태 변경 로직에서 특히 중요**하다. 실패했는데 일부만 반영된 상태라면 세이브 파일이 깨지거나 인벤토리 수량이 꼬이는 심각한 버그로 이어진다.

```cs
// 나쁜 예: 중간에 예외가 나면 아이템은 이미 빠졌는데 골드는 안 늘어난 상태로 남음
public void SellItem(Item item)
{
    _inventory.Remove(item);       // ① 아이템 제거 완료
    _gold += CalculatePrice(item); // ② 여기서 예외가 나면? -> 아이템은 사라졌는데 골드는 그대로
}

// 좋은 예: 실패 가능성이 있는 계산을 먼저 끝내고, 상태 변경은 마지막에 한 번에
public void SellItem(Item item)
{
    int price = CalculatePrice(item); // 실패할 수 있는 부분을 먼저 (예외 나도 상태 변경 없음)
    _inventory.Remove(item);          // 여기부터는 실패 가능성이 없다고 확신되는 부분만
    _gold += price;
}
```

## catch 후 예외를 다시 발생시키는 것보다, 예외 필터가 낫다
- 특정 조건일 때만 예외를 처리하고 싶다면, `catch`로 일단 잡은 뒤 조건을 체크해 다시 `throw`하는 것보다는 `catch (Exception ex) when (조건`) 형태의 예외 필터를 쓰는 게 스택 추적을 보존하고 성능도 낫다.

- 유니티
	- 특정 상황(네트워크 재시도, 특정 에러 코드 처리)에서 유용하다.
	- 예외 필터를 쓰면 조건이 안 맞을 때는 애초에 `catch` 블록에 진입조차 하지 않기 때문에 원래 예외 스택 추적이 그대로 보존된다.

```cs
// 나쁜 예: catch에서 조건 체크 후 다시 던짐 -> 스택 추적이 이 지점부터 새로 시작된 것처럼 보임
try
{
    LoadRemoteConfig();
}
catch (WebException ex)
{
    if (ex.Status != WebExceptionStatus.Timeout)
        throw; // 스택 추적 정보가 살짝 손실될 수 있음(다만 throw만 쓰면 보존은 됨, rethrow with new throw ex; 가 진짜 문제)
    RetryWithBackup();
}

// 좋은 예: 예외 필터로 조건을 아예 catch 진입 조건에 포함
try
{
    LoadRemoteConfig();
}
catch (WebException ex) when (ex.Status == WebExceptionStatus.Timeout)
{
    RetryWithBackup(); // 타임아웃일 때만 이 블록에 진입, 그 외엔 상위로 자연스럽게 전파
}
```

## 예외 필터의 다른 활용 예를 살펴보라
- 예외 필터`when`는 조건부 `catch` 뿐만 아니라 부수 효과(로깅 등)를 원본 예외 흐름에 영향 없이 끼워넣는 용도로 활용할 수 있다.

- 유니티
	- 크래시 리포팅, 원격 로그 수집 등을 붙일 때 유용한 패턴이다.
	- `when` 절 안에서 로깅만 하고 `false`를 반환하면 예외는 로그만 남긴 채 그대로 위로 전파된다.

```cs
bool LogAndContinue(Exception ex)
{
    Debug.LogError($"예외 발생, 로그 남기고 계속 전파: {ex}");
    return false; // false를 반환하면 이 catch는 "처리하지 않은 것"으로 취급되어 예외가 계속 위로 전파됨
}

void RunGameLogic()
{
    try
    {
        DoRiskyOperation();
    }
    catch (Exception ex) when (LogAndContinue(ex))
    {
        // 이 블록은 절대 실행되지 않음 (필터가 항상 false 반환)
        // 하지만 필터 자체는 실행되므로 로깅은 확실히 일어남
    }
    // 로그를 남긴 뒤에도 예외는 그대로 상위로 전파되어, 원래 예외 처리 흐름(예: 크래시 리포터)이 그대로 동작
}
```