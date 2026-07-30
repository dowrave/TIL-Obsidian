
```cs
private Dictionary<Type, EnemyState> _states = new();

_states[typeof(EnemyMoveState)] = new EnemyMoveState(this);
_states[typeof(EnemySpawnState)] = new EnemySpawnState(this);

public void ChangeState<T>() where T : EnemyState
 {
	_currentState?.Exit();
	_currentState = _states[typeof(T)];
	_currentState.Enter();
}
```

- `typeof()` 
	- **컴파일 타임**에 타입을 대표하는 `Type` 객체를 가져오는 연산자. 
	- 함수 호출이 아니라 C# 연산자다.
- `System.Type`
	- 타입에 대한 정보를 담고 있는 객체. 
	- 이것 자체도 클래스이자 객체이므로, 한 개의 타입에 대해 항상 같은 `Type` 인스턴스가 반환된다.
	- `리플렉션`의 기본 단위로, 이 타입이 무엇인지를 런타임에도 알 수 있게 해주는 객체다.
	- 비유하면
		- **`EnemyState` : 설계도.** "메서드와 필드를 가졌다"는 정의
		- **`typeof(EnemyState)` : 설계도에 대한 설명서.** 네임스페이스, 부모 클래스, 메서드 목록 등의 정보를 담고 있다. `Type` 클래스.

> 여기까지만 알아놔도 무방. 더 궁금하거나 정보가 부족하다고 느끼면 채우자.
> 아래는 디테일.

## Type 객체 생성 시점에 대한 디테일한 설명
- 인스턴스가 없어도 `Type` 객체는 존재할 수 있음
- **`Type` 객체는 타입 로딩 시점에 생성**된다.
	- `C#/.NET`은 컴파일 시 클래스가 `IL(중간 언어)` 코드와 메타데이터 형태로 어셈블리`dll/exe`에 저장된다. 이 메타데이터에는 클래스 이름, 필드 ,메서드, 부모 클래스 등의 정보가 이미 들어 있다 - **소스 코드를 컴파일하는 순간 이미 존재**한다.
	- **타입 로딩** : 프로그램이 실행되며 특정 타입이 처음 "사용"되는 시점에, `CLR(Common Langauage Runtime)`이 그 메타데이터를 읽어서 런타임 메모리에 `Type` 객체를 만들어 올린다.
		- 트리거 시점은 다양하다
			- `typeof(X)` 호출
			- `new X()`로 인스턴스 생성(타입 로딩 후, 인스턴스가 생성됨)
			- 정적 필드/메서드를 처음 참조
			- 리플렉션으로 타입 조회(`Assembly.GetType("..."))`)
		- 위 예제 `_states[typeof(EnemyMoveState)] = new EnemyMoveState(this);`에서는 `typeof()`시점에서 `Type` 객체가 얻어진다. 
