

# 1. 유니티의 `null`체크는 다르게 동작한다.
- `UnityEngine.Object`를 상속받는 모든 객체(`GameObject, MonoBehaviour`)의 `null` 체크가 특별하게 구현되어 있다. 내부적으로 오버라이딩 된 `== null` 연산자가 사용되며, 단순한 `null` 비교가 아니라 객체가 이미 파괴되었는가?를 확인하는 과정이 추가된다.

```cs
GameObject obj = someGameObject;
Destroy(obj); 

if (obj == null) // true
```
> `obj`라는 변수를 `null`로 설정하지 않았음에도,  **유니티의 오버라이딩된 연산자 `obj == null`은 `true`를 반환한다.**

그런데 여기서 문제는, **`C#` 레벨에서는 `obj`가 `null`일 수 있다**는 것이다. 그래서 아래 같은 상황이 발생할 수 있다.
```cs
if (obj != null) {
	// obj가 사용 가능함을 가정하지만, 파괴된 객체일 수 있음
}
```

- 유니티의 **null 체크는 추가적으로 엔진에서의 파괴 여부를 검사**한다
	-  `== null` 은 단순한 포인터 비교가 아니기 때문에, 다량의 `null` 체크는 성능에 영향을 줄 수 있게 된다.

# 2. 개선 방향

1. `ReferenceEquals()` 사용
```cs
if (ReferenceEquals(obj, null))
{
    Debug.Log("obj는 진짜 null입니다.");
}
```
> 해당 `obj`가 `null`인지에 대한 여부만 검사하고 싶을 때 사용함

2. **`obj != null` 체크를 줄이도록 구조 변경**
	- 반복적으로 사용되는 메서드(예시: `Update`)에서 `null`체크를 하지 않고, 특정 순간에만 검사하도록 함
```cs
private void OnDisable() 
{
    myObject = null;  // 객체가 비활성화될 때 null로 설정
}
```

3. 에디터에서 할당하는 필드일 경우, `required`을 붙일 수도 있다.
```cs
    [UnityEngine.Serialization.Required]
    private GameObject _player = default!;
```