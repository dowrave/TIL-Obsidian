
- `Time.timeScale` : 게임 시간이 흐르는 속도
	- 디폴트는 1로, 현실의 1초와 게임의 1초가 동일하다. 

- 값의 의미
	- 1 : 기본값, 정상 속도
	- 0 : 게임 일시 정지
	- 0.5 : 절반 속도
	- 2 : 2배 속도

- 영향을 받는 값
	- `Time.deltaTime`
	- `Update(), FixedUpdate()` 호출 간격
		- `0`일 경우 `FixedUpdate()`는 호출되지 않음
	- 애니메이션
	- 물리 시뮬레이션

- 영향을 받지 않는 값
	- `Time.realtimeSinceStartup`
	- `Time.unscaledDeltaTime`
	- `LateUpdate()`
	- 코루틴 - `WaitForSecondsRealtime()`

- `Time.timeScale`은 전역적으로 적용된다 -> 세밀한 제어가 필요한 경우, 별도의 로직을 구현해야 한다.