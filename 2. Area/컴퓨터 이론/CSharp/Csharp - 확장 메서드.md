#Csharp 

>[!note]
>- **`static class` 내부의 `static` 메서드의 1번째 매개변수 앞에 `this`가 붙는다면, 1번째 매개변수의 타입에 대한 확장 메서드를 정의할 수 있다.** 
>	- 해당 타입을 정의하는 스크립트에 직접 접근하지 않고도!

## 상황
- `IReadOnlyList<T>`에는 `IndexOf`이 없다. 
- 이 이슈를 해결할 방법을 제미나이에게 물어봤는데, 아래의 방법을 알려줬음.
```cs
using System.Collections.Generic;

public static class ListExtensions
{
    // IReadOnlyList에 IndexOf 기능을 추가
    public static int IndexOf<T>(this IReadOnlyList<T> list, T item)
    {
        for (int i = 0; i < list.Count; i++)
        {
            if (EqualityComparer<T>.Default.Equals(list[i], item))
                return i;
        }
        return -1;
    }
}
```
- **`IReadOnlyList`의 스크립트를 직접 수정하지 않고 기능을 추가할 수 있다**는 얘기인데, 어떻게 가능할까?


## 내용

### 1. `this` 키워드

- 확장 메서드를 만들기 위한 2가지 규칙
1. `static class` 내부의 `static`일 것
2. 첫 번째 매개변수 앞에 `this`를 붙일 것

이것만으로 `this IReadOnlyList<T> list`로 들어올 때, 컴파일러는 `IReadOnlyList<T>`에서 `.IndexOf()` 형태로 호출하도록 연결해주는 규칙이 추가됐음을 인지할 수 있다.


### 2. 컴파일러의 눈속임

- 위 코드는 곧바로 이렇게 쓸 수 있다.
```cs
int index = Combat.CurrentActionableGridPos.IndexOf(targetPos);
```

- 컴파일러가 이를 번역할 때는, `static` 클래스의 함수에 1번째 인자로 해당 리스트를 집어넣는 형태로 자동 변환한다.
```cs
int index = ListExtensions.IndexOf(Combat.CurrentActionableGridPos, targetPos);
```

> 즉 **static 클래스 호출에 관한 자동변환이 이뤄진다**는 것. 

### 3. 내부 동작 로직
- `IReadOnlyList`는 `IndexOf`은 없고, 총 몇 개인지`Count`와 인덱서 기능은 갖고 있다.
- 확장 메서드 내부에서 그 기능들을 이용해 직접 인덱스를 찾는 로직을 구현한 것이 된다.
```cs
public static int IndexOf<T>(this IReadOnlyList<T> list, T item)
{
    for (int i = 0; i < list.Count; i++) // Count 속성 사용 가능
    {
        // EqualityComparer는 제네릭(T)에서 '==' 대신 값을 비교할 때 쓰는 가장 안전한 C# 표준 방식
        if (EqualityComparer<T>.Default.Equals(list[i], item)) 
            return i; // 값을 찾으면 해당 인덱스(i) 반환
    }
    return -1; // 끝까지 못 찾으면 -1 반환
}
```

## 역사

- 확장 메서드는 C# 3.0 (2007)에 처음 등장했다. 이전엔 아래의 문제가 있었음.

### 탄생 배경
#### 1. 수정할 수 없는 클래스들

- 예시 
	- C#의 `string` 클래스, 유니티의 `Transform` 클래스에 나만의 유용한 함수를 추가하고 싶다

- 원본 소스 코드가 없어서 직접 수정할 수 없다.
- `string` 등은 `sealed`(상속 불가)로 막혀 있어서 상속받아 새로운 클래스를 만들 수도 없다.

#### 2. 불편한 유틸리티 헬퍼 클레스

1번 이슈 때문에, 아래처럼 구현하곤 했다.

```cs
string myText = "Hello";
bool isEmail = StringHelper.IsValidEmail(myText);
```

기능상 문제는 없지만, 객체지향적이지 않고 코드를 읽을 때 직관성이 떨어진다. `MyText.IsValidEmail()`처럼 쓰는 게 훨씬 자연스럽다.

#### 3. LINQ의 도입
- LINQ를 도입하려고 할 때 가장 큰 문제가 생겼다. 모든 컬렉션`IEnumerable`에서 사용할 수 있게 하려면 인터페이스를 수정해야 했는데, 이 경우 기존의 C# 코드들이 에러를 발생시켰기 때문이다.

#### 따라서
- 외부에서 메서드를 주입하는 것처럼 보이는 식으로 만들 필요가 있었다. 여기서 확장 메서드가 탄생했다.

### 동작 원리
위에서 설명한 것과 동일하다. 전역 클래스의 전역 메서드의 1번째 매개변수에 `this`와 함께 타입을 전달하면, 컴파일러에서는 해당 타입에 대한 메서드를 호출하는 것처럼 바뀐다.
```cs
public static class MyExtensions
{
    // static 클래스 내부의 static 메서드 + 첫 번째 매개변수에 this
    public static void ResetPos(this Transform transform)
    {
        transform.position = Vector3.zero;
    }
}

// 개발자 작성 코드
transform.ResetPos();

// 컴파일러가 변경해서 전달
MyExtensions.ResetPos(transform);
```
