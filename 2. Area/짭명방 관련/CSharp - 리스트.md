- 리스트는 **가변적인 길이를 갖는 배열**로 보면 된다. (파이썬이 아니므로) 한 가지의 타입만을 받을 수 있는 건 배열과 동일하다.

- 초기화할 때 크기를 지정할 수 있다
```cs
currentSquad = new List<OperatorData>(6);
```
**이 때, 리스트에는 아무 것도 없다.** 사용하기 위한 공간만을 할당해둔 상태이다.

> 1. 따라서 이 상태에서 인덱스 접근을 하면 `IndexOutOfRangeException` 오류가 발생한다.
> 2. 추가로, **지금처럼 아무 것도 없는 상태에서 `currentSquad.Add(element)`를 하면 0번 인덱스에 값이 들어가며, 길이도 그대로 유지**된다.

- 이를 구분하기 위한 프로퍼티로, `Capacity`와 `Count` 가 있다.
```cs
list.Count // 배열에 저장된 "원소의 수"
list.Capacity // 배열이 실제로 메모리에 할당된 "크기"
```
> `Add` 메서드는 `Count`를 인덱스로 사용해서 원소를 추가하는 과정이다. 즉 아무 것도 없다면 `0번 인덱스`에 원소를 추가하는 것이다.



- 즉 위 과정은 최적화를 위해 초기에 사용할 공간을 잡아놓는 과정인 것이며, 실제로 리스트에 값을 할당하려면 아래의 방법들을 사용할 수 있다.
```cs
currentSquad = new List<OperatorData>(MaxSquadSize);
for (int i = 0; i < MaxSquadSize; i++)
{
    currentSquad.Add(null);
}

// 방법 2: Enumerable.Repeat 사용
currentSquad = Enumerable.Repeat<OperatorData>(null, MaxSquadSize).ToList();

// 방법 3: 배열을 만들고 리스트로 변환
currentSquad = new OperatorData[MaxSquadSize].ToList();
```

