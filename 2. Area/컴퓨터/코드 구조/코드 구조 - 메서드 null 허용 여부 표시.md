>[!question]
>메서드의 파라미터에 `null`을 달지 말지 여부는 어떻게 표시해둘까?

## 답변) 파라미터에 달아두셈
```cs
private void PopulateContent(SaveSlotMetadata metadata = null)
```
- 만약 `= null`이 없다면 코드만 봤을 때 '이 파라미터에 null을 허용하는지 아닌지 구분이 가지 않는 상황'이 생길 수 있음.


## 추가 지식

### NRT(Nullable Reference Types)
- 파라미터에 `?`을 달아서 `Nullable`임을 표시하는 것.
- 기본적으로 꺼져 있고, `.csproj`에 명시해두면 된다.
```
<Nullable>enable</Nullable>
```

```cs
private void PopulateContent(SaveSlotMetadata? metadata = null)
```

>[!note]
>- **진행 중인 유니티 프로젝트에서 켜는 건 권장하지 않음**
>1. 참조 타입은 원래부터 `null`이 될 수 있음
>2. 유니티에서는 실패 시 `null`을 반환하는 API들이 대부분 `non-null` 시그니처로 되어 있음. NRT를 켜는 순간 관련 없는 경고가 대량으로 쏟아짐.
>3. `[SerializeField]`는 생성자 시점엔 `null`이나 실행 시점엔 채워지는데, `NRT`는 유니티 특유의 초기화 흐름을 알지 못함
>- 단 순수 C# 클래스에 대해서는 NRT의 이득을 온전히 받을 수 있음.


#### `NRT`가 꺼져 있는 경우와의 차이점
```cs
private void PopulateContent(SaveSlotMetadata metadata = null)
```
1. `NRT`가 꺼져 있는 경우 : 경고 없음
2. `NRT`가 켜져 있는 경우 : `null`을 허용하지 않는데 `null`을 왜 넘기려고 하냐는 경고가 뜬다. `CS8625`

```cs
private void PopulateContent(SaveSlotMetadata? metadata = null)
```
1. `NRT`가 꺼져 있는 경우 : 경고 없음(`?`이 무시됨)
2. `NRT`가 켜져 있는 경우 : 경고 없음. 의도가 명시되어 있음.



### `Nullable<T>`
- 값 타입 `T`에 `null`을 사용하기 위한 구조체.
	- **원래 값 타입은 `null`이 될 수 없는데, "값이 없음"이라는 상태를 표현해주기 위한 구조체**다.
- `NRT`와는 같은 기호 `?`을 쓰지만, 컴파일러 레벨과 런타임 레벨에서의 성격이 아예 다르다.
	- `Nullable<T>`라는 구조체로 컴파일됨
	- 값에 접근하려면 `.Value`, 값이 있는지 여부는 `HasValue`로 체크. 