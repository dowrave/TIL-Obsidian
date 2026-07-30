#Pigments 

- 핵심 
	- `LP` Filter의 Cutoff을 높은 곳에서 낮은 곳으로 낮추는 것
		- 별도의 엔벨로프를 연결해서 구현
	- 볼륨 엔벨로프는 `Sustain`을 0으로 만들어서 꾹 누르더라도 소리가 지속되지 않게 할 것

![[Pasted image 20260328223532.png]]
- 강의에서 `Decay`와 `Release`를 맞추라는 얘기가 여러 번 나왔는데, 저 아이콘으로 동기화할 수 있음

- 2개의 오실레이터를 켜고, 하나에는 FM을 거는 방식으로 텍스쳐를 줄 수 있음
![[Pasted image 20260328223715.png]]
> - 실험했을 때 2번째 파동의 주파수가 어느 정도 높을 때(=FM Modulation Amount값이 어느 정도 높을 때) 리얼한 소리가 남.
> - 약간 애매하게 주면 디스토션처럼 지지직거리는 소리가 난다.

- 강의에서는 `Amount`에 3번째 엔벨로프를 걸어줌
	- `Attack`을 0으로 해서 초기에 증가하는 효과를 없앰
	- `Decay, Release`는 기존 엔벨로프들보다 짧게
	- `Amount`의 초깃값을 살짝 올려줌

- 어쨌든 디스토션 질감이 안 들리고 깔끔하게 나면 된다. 디테일을 어떻게 챙기는지를 적어둠.

- 랜더마이즈
	- 모드 : `Sample & Hold`
	- `Filter - Resonance`
	- `Fine Tuning`
	- `Modulator - Amount`

- `Cutoff`에도 랜더마이저를 걸었는데, 이 경우 마이너스값도 반영되므로 필터 컷오프가 항상 +에서 -로 이동해야 하는 `Pluck` 사운드의 기저에서 벗어남
- 이 경우 `Func`을 활용할 수 있음
![[Pasted image 20260328230913.png]]
> `Func`의 모양은 좌상단 -> 우하단 이동
> - `Cutoff` 자체에 `SideChain`을 `Random 1`로 연결해서 걸 수 있다. `Function`의 최댓값을 `Random 1`의 값을 가져와서 사용하겠다는 의미임
> - 근데 `Randomizer - Sample & Hold`는 `Polarity`를 고정할 수 없음 -> `Rand 1`의 샘플로 `White Noise` 대신 `Rand 2 - Randomizer(Polarizer : +)`를 연결하는 방식으로 설정 가능


- 이후 딜레이를 걸어봤으나 소리 자체가 안 예뻐서 다시 원래 사운드 수정

- 이것저것 합니다. 
	- 디스토션 - 릴레이 - 리버브 순으로 걸고, 
	- 디스토션에 엔벨로프 3(모듈레이터 Amount에 걸었던)을 디스토션의 Dry/Wet 및 Drive에 걸고.. 