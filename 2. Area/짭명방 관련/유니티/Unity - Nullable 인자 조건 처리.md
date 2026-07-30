- 내 프로젝트에서 뽑아냄
```cs
public bool? IsOnSelectionSession => MainMenuManager.Instance?.OperatorSelectionSession;
```
이라는 **`nullable` 프로퍼티**가 있다고 하자.

그리고 아래의 조건문이 있다.
```cs
if (!IsOnSelectionSession)
```

이걸 작성한 목적은 `bool`인 변수가 `false`일 때에만 동작시키기 위함이었음.

근데 저렇게 작성하면 `암시적으로 bool?을 bool로 변환할 수 없다`라는 빨간 밑줄이 뜬다. 

왜냐하면 `!`은 논리 부정 연산자인데, `bool?`이 가질 수 있는 상태는 `true, false, null` 3가지이다. **`false`인지 `null`인지가 명확하지 않기 때문에 저런 제한을 걸어둔 것**이다.

따라서 **`nullable` 변수에 대해서는 논리 부정 연산자 `!`을 쓸 수 없다.** 

위 코드를 내 목적에 따라 고치면
```cs
if (IsOnSelectionSession == false)
```
처럼 처리해야 정확하다. `null, true`일 때는 실행하지 않겠다는 의미가 명확해지기 때문이다.