
>[!done]
>1. **`Burst`는 0초에 파티클을 바로 생기게 할 수 있다.** 
>2. **`Rate over Time`은 1보다 작은 소수값으로도 설정할 수 있다.** 
>	- 1개의 파티클을 유지하고자 한다면 `Start Lifetime * Rate over Time = 1`을 유지하도록 값을 설정하면 됨
>3. **`Rate over Time`으로 적은 수의 파티클을** 다룰 경우, 0초에 파티클이 나타나지 않을 수 있다.
>	- **`Prewarm` 옵션으로 해결**한다. (`Looping`이 켜져 있어야 함)
>		- 단 이 경우 파티클 시스템을 멈추고 `Playback Time`을 다시 테스트할 때 안 보이는 현상이 있다.
>	- `Burst`로 구현해도 가능은 한데 파티클이 겹치는 문제가 있음. 이건 왜인지는 모르겠다.

- 공통 버프 작업을 다시 진행 중인데, `Beam` 이미지가 커졌다가 작아졌다가 하는 식으로 바닥 파티클을 구현하는 중.

![[Pasted image 20250805134300.png]]

여기서 `Size over Lifetime`, `Rate over Time`, `Lifetime` 관련 이슈가 있었다.
- `Rate over Time` : 1초당 생성하는 파티클의 갯수
- `Size over Lifetime` : `Lifetime` 동안 파티클의 크기 변화
- `Lifetime` : 개별 파티클의 지속 시간

`Rate over Time`이 **1초당**에 해당하는 개념이므로, 만약 `Lifetime`이 1초보다 길다면 바닥의 파티클이 겹치는 문제가 발생했다. 그렇다고 `Lifetime`을 줄이자니 `Size over Lifetime`의 변화 속도가 너무 빨라지는 문제도 있다.

제미나이님께 물어보니, **`Rate over Time`은 소수로도 설정할 수 있다.** 예를 들어 `Start Lifetime = 2s`, `Rate over Time = 0.5`로 설정한다면 파티클은 2초에 한 번 생긴다. 

이렇게 설정하고 테스트해보면 파티클 시스템을 재생해도 0 ~ 2초까지는 이펙트가 보이지 않는다.

이 문제는 `Prewarm` 옵션을 켜는 것으로 해결할 수 있다.
- 단순히 `Start Delay`가 있을 때 이펙트가 자연스럽게 실행되도록 할 때만 쓰는 기능인 줄 알았는데, 이럴 때도 쓸 수 있다.

여기서 파티클 시스템의 `Playback Time`을 다시 조작했을 때, **0 ~ 2초 사이의 파티클이 사라지는 현상**이 있다. 이건 **`Playback Time`을 조작할 때 `Prewarm`에 의한 계산은 고려하지 않기 때문이다.**

