#Csharp 
## 정의
- **람다식, 혹은 익명 함수가 자신이 정의된 곳의 바깥 변수를 기억**하는 것

```cs
int score = 10;
Action printScore = () => Debug.Log(score); // 람다가 바깥의 score를 참조
printScore(); // 10
```

`score`라는 변수를 함수가 직접 정의하지 않았음에도 그걸 쓸 수 있는 상황이다. 이렇게 자기 몸 바깥의 변수를 안고 있는 함수를 `클로저Closure (닫혀서 감싼 것)`라고 부른다. 

### 이게 왜 신기함?
- 일반적인 함수는 자기 안에서 선언한 지역변수만 쓸 수 있고, 함수가 끝나면 지역변수는 사라진다.
- 하지만 람다는 다르다.
```cs
Action MakeCounter()
{
    int count = 0; // 이 메서드의 지역변수

    Action increment = () => {
        count++; // 바깥(MakeCounter의) 지역변수를 참조
        Debug.Log(count);
    };

    return increment; // MakeCounter는 여기서 끝나는데...
}

Action counter = MakeCounter();
counter(); // 1
counter(); // 2
counter(); // 3
```

- `MakeCounter()`는 실행이 끝나서 사라져야 하므로 `counter()`는 모두 1이 되는 것이 기대되는데, 서서히 값이 증가한다.

## 핵심 : 값을 복사하는 게 아니라 변수를 참조하는 것
- **람다는 값을 복사해서 가져가지 않고, 변수가 저장된 메모리 칸을 계속 보고 있다.**
- 그래서 아래 같은 버그가 난다. 
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

### 람다의 동작 원리
1. `int i`가 반복문 밖에서 선언되는 경우
```cs
for (int i = 0; i < 3; i++)
{
    actions.Add(() => Debug.Log(i));
}

// 위 코드를 컴파일러는 아래처럼 만든다. 개념적으로 보면 됨.
var box = new ClosureBox(); // 상자는 루프 시작 전, 딱 1번만 생성됨
box.i = 0;
for (; box.i < 3; box.i++)
{
    actions.Add(() => Debug.Log(box.i)); // 세 람다 모두 "같은 box"를 참조
}
```

2. `captured`로 반복문 내의 요소를 잡는 경우
```cs
for (int i = 0; i < 3; i++)
{
    int captured = i;
    actions.Add(() => Debug.Log(captured));
}

// ---
for (int i = 0; i < 3; i++) // i에 대한 클로저는 생기지 않는다. i를 참조하는 람다 함수가 없으니까. 
{
    var capturedBox = new ClosureBox(); // 몸체 진입할 때마다 "새 상자"를 만듦!
    capturedBox.captured = i;
    actions.Add(() => Debug.Log(capturedBox.captured)); // 이번 반복의 람다는 "이번에 새로 만든 상자"만 봄
}
```

>[!note]
>1. **컴파일 타임 : 값 변수가 람다에 캡처되는지 아닌지를 판단함.** 
>2. 런타임
>	- **캡처되는 변수라면 해당 변수에 대한 객체를 힙에 만듦**
>	- 캡처되지 않는다면 아무 처리하지 않음

### 추가 정보
- 람다 하나에 여러 변수를 참조한다면, 이들은 여러 객체로 흩어지지 않고 하나의 객체 안에 들어간다.
```cs
int a = 1, b = 2;
Action act = () => Debug.Log(a + b); // a, b 둘 다 하나의 상자 안에 필드로 같이 들어감
```
