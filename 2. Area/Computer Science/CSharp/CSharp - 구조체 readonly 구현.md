- [[자료구조 - 3. 순차 자료구조와 선형 리스트]]에서 희소 행렬을 압축한 행렬을 다루다가 든 의문.
- **가장 쉬운 예시는 `Vector3`이다.** 얘의 값을 갱신할 때는 기존 벡터의 특정 필드에 접근해서 갱신하는 게 아니라, 새로운 벡터를 만들어서 갱신하는 걸 많이 봤을 거임.

## 결론
- `mutable struct` 선언일 때 발생하는 문제점 
	1. 필드/프로퍼티 값 할당 / 수정 시
		1. 필드에는 직접 접근해서 할당할 수 있다.
		2. 메서드/프로퍼티는 임시 복사본을 만드는 개념이다. 
			- 곧 없어질 값에 값을 넣는 개념이라 컴파일 시점에 오류 발생.
	2. 방어적 복사 : `readonly` 필드로 쓰였을 때 구조체 자신을 수정하는 mutating 메서드가 실행되면 **조용히** 복사본을 만들어서 실행하고 원본에 반영되지 않는다. 이는 성능 저하를 유발한다.

- **`readonly struct`로 선언하면 자기 자신을 수정하는 메서드`mutating method`를 정의할 수 없게 된다.** 
	- `readonly`에서는 프로퍼티도 게터만 정의할 수 있고 세터는 정의할 수 없음.
	- 자신을 수정하는 건 아예 새로운 구조체를 만들어 넣는 식으로 구현한다.

## 디테일
- 아예 예제를 통째로 보자.
```cs
struct Counter
{
    public int value;              // 필드
    public void Increment()        // mutating 메서드 (자기 자신을 수정)
    {
        value++;
    }
}
```
- `mutatable struct`, 즉 바뀔 수 있는 구조체다. 
- 자신을 바꾸는 `mutating` 메서드를 가졌다.

### 1. 필드와 프로퍼티에 따른 차이
```cs
class Container
{
    public Counter counterField;               // 필드
    public Counter counterProperty { get; set; } // 프로퍼티
}

List<Counter> counterList = new List<Counter>();
counterList.Add(new Counter());

Container c = new Container();

// 1. 필드 접근 - 원본 주소를 직접 구하므로 문제 없음
c.counterField.Increment(); // 클래스 내부 객체의 field값 1 증가.
c.counterField.value = 10; // 수정 가능

// 2. 프로퍼티 접근
c.counterProperty.Increment(); // 컴파일 에러 CS1612
c.counterProperty.value = 10; // 컴파일 에러 CS1612

// 인덱서도 프로퍼티와 동일
counterList[0].Increment(); // 컴파일 에러 CS1612
```
- 프로퍼티, 인덱서 접근에서 오류가 나는 이유는 `get_counterProperty()`가 반환한 임시값이라서 주소가 없기 때문이다. 원본을 고치는 게 아니라 임시 복사본을 고치는 개념이다.


### 2. 방어적 복사 - readonly "필드"로만 쓰였을 때
```cs
class Container2
{
    public readonly Counter counterReadonlyField = new Counter();

    public void Test()
    {
        counterReadonlyField.Increment(); // ✅ 컴파일 됨!
        Console.WriteLine(counterReadonlyField.value); // 항상 0!
    }
}
```
- 메모리, 성능 관점의 개념이다.
- C# 컴파일러는 `readonly` 상태인 필드에 접근할 때, 해당 구조체가 내부 값을 바꿀지 모른다는 의심을 한다. 그래서 원본 파괴를 막기 위해 **개발자 몰래 숨겨진 복사본을 생성**한다.
- 즉, `counterReadOnlyField`의 복사본을 생성하고 거기서 값을 1 올리지만, 실제 원본에는 반영되지 않는 현상이 발생한다.
- 따라서 매 프레임 `UpdatePosition()`이 호출될 때마다 불필요한 메모리 복사 연산이 발생해 성능이 떨어진다.
-  **겉보기에는 정상 동작하는데(에러 발생 X) 아무 효과가 없다는 것**

- 이게 발생할 수 있는 상황
	- `readonly` 필드의 자신을 바꿀 수 있는(mutating) 메서드 호출(방어적 복사)
	- `in` 매개 변수로 받은 `struct`의 mutating 메서드 호출
	- `foreach`로 순회 중인 변수의 mutating 메서드 호출
	- `readonly` 지역 변수(C#에는 없음. 개념상)의 mutating 메서드

### 3. readonly struct로 구현하면
```cs
readonly struct Counter
{
    public readonly int value; // 필드
    // public void Increment() { value++; } // ❌ 컴파일 에러!
    // "readonly struct의 멤버는 자기 필드를 수정할 수 없습니다"
    public Counter(int v) { value = v; }
    public Counter Incremented() => new Counter(value + 1); // 새 값을 반환
}

// 사용
counterReadonlyField = counterReadonlyField.Incremented(); // 통째로 재대입
```
- **자기 필드를 바꾸는 메서드 자체를 정의할 수 없게 된다.**
	- 문제 1은 애초에 수정 가능한 메서드를 정의할 수 없게 됐다는 점, 프로퍼티 또한 `readonly`로 인해 구현할 필요가 사라졌다는 점 등이 있다.
	- 문제 2는 컴파일러가 "이 구조체는 자기 자신을 절대 수정할 수 없다"는 걸 알게 됐으니 방어적 복사 자체가 필요 없어진다. 복사가 생략되고, 조용한 버그도 발생하지 않는다. 

### 4. (고수용?)예외
- **대부분의 경우 `readonly struct`가 정답**임.
- `Mutable Struct`가 더 유용한 경우가 있다.
- 성능이 중요한 상황에서 배열 / NativeArray 안의 원소를 제자리에서 수정할 때가 그렇다.

- 파티클이 대표적인 예시.
```cs
struct Particle // 일부러 mutable
{
    public Vector3 position;
    public Vector3 velocity;
}

NativeArray<Particle> particles;

// Job 안에서
void Execute(int index)
{
    var p = particles[index];
    p.position += p.velocity * deltaTime;
    particles[index] = p; // 복사해서 넣고 빼는 패턴
}
```

- 이런 게 매 프레임 수백만 개 돌아가므로, `struct`를 계속 일일이 만들어 넣는 것보다 `ref` 인덱서나 포인터로 직접 필드를 바꾸는 게 더 빠른 경우가 있다.
- 유니티에서는 `NativeArray<T>.this[int]` 나 `Span<T>` 기반 코드에서 이런 게 흔하다고 함.