- 1개의 메서드만을 갖는다.
```cs
bool IsRaycastLocationValid(Vector2 screenPoint, Camera eventCamera);
```

1. `Vector2 screenPoint`
- 현재 마우스 포인터 / 터치 지점의 스크린 좌표. `Input.mousePosition`과 거의 동일한 값이다.

2. `Camera eventCamera`
- 해당 캔버스를 렌더링하고 이벤트를 발생시키는 데 사용되는 카메라.
- `Screen Space - overlay` 캔버스의 경우 `null`이 될 수 있음
- `Screen Space - Camera`나 `World Space` 캔버스라면 해당 캔버스에 설정된 카메라가 전달된다.
- 카메라 정보는 스크린 좌표를 월드 좌표로 변환하는 등의 작업에 필요할 수 있다.

### 반환값
- **`true` 시 이 요소를 상속받은 객체의 클릭 이벤트** 처리
- **`false` 시 이 요소를 무시하라**는 처리

