
- [유튜브 강의](https://www.youtube.com/watch?v=taMp1g1pBeE)

- 그래프 설정 
	- `Unlit`, `Opaque`, `Both`, `Alpha Clipping : True`
	- 참고) Allow Material Override를 해도 Alpha Clip Threshold가 나오지만, 이 경우는 실습이 제대로 되지 않을 수 있음. Alpha Clipping : True로 작업하는 게 더 정확함

#### 1. 정말 간단하게 구현하기
```
Simple Noise -> Alpha 연결

Float 생성, Alpha Clip Threshold 값에 연결하고 슬라이더로 연결해서 테스트
```
이러면 생성한 `Float` 값에 따라 생성한 셰이더가 사라지는 효과가 나타날 거임

> 구현 방식 : 모든 픽셀에 랜덤한 노이즈를 주고, 그 노이즈값을 알파로 이용해서 `Alpha Threshold` 값에 따라 그 위치를 보여주거나 보여주지 않는 식
> **랜덤한 위치에 `[0, 1]`의 값을 부여하고 경계선을 조정해서 보여주거나 보여주지 않는 식이라, 생각보다 자연스러운 효과가 나타난다.** 물론 여기까지만 하면 시간에 따른 효과는 구현하지 않음.
#### 2. 더 깊게 들어가기
우선 Graph Settings의 Unlit 을 Lit로 바꿈

> `Lit`의 마스터 노드의 `Emission`을 사용할 거임
> - 물체가 자체적으로 빛을 발산하는 효과. HDR의 Intensity로 밝은 색상을 설정할 수 있다.

1. **`Time - Sine Time`** 을 `Remap`의 인풋으로 넣음
> Remap 결과로 나오는 `Out` 값은 Sine의 `-1, 1`을 `0, 1`로 다시 매핑함.

2. `Remap`의 아웃풋을 `Alpha Clip Threshold`로 넣음
> 1에 의해, Alpha Clip Threshold는 `[0, 1]`을 오가는 사인함수가 된다. 

여기까지 구현하면 시간에 따라 나타나고 사라지는 효과가 구현됨

3. `Fragment - Emission`에 연결할 내용
```
1) Step 노드를 생성
2) Edge(1)에는 Simple Noise의 아웃풋을
3) In(1)에는 위에서 작업한 Remap + Add 노드를 연결함
```
- 이 Step 노드는 Simple Noise에 대해, Remap + Add 값을 넘는 부분은 나타나고 넘지 않는 부분은 나타나지 않음. 
- 이 `Step` 노드의 `Output`은 `Fragment - Emission`에 연결된다.

> 3번에서 구현된 내용은, Dissolve 효과가 구현되는 중에 그 경계선을 나타내는 표현을 추가하는 것이다. 구체적으로 들어가면 아래와 같다.
```
Simple Noise 값이 0.5인 지점이 있다고 하자. Add 노드에서 더해지는 값은 0.03 정도라고 가정한다. Alpha Clip Threshold를 ACT라고 하겠다.

크게 3가지 상황이 있다.
1. Simple Noise < ACT
2. Simple Noise > ACT & Simple Noise < ACT + 0.03
3. Simple Noise > ACT + 0.03

1번 상황의 경우, 알파 조건보다 작으므로 해당 부분은 나타나지 않는다.
2번 상황의 경우, 나타나지만 Step 조건보다 작다. 따라서 Emission에는 해당되지 않는다.
3번 상황의 경우, Emission 까지 해당된다.
```

결국 따지면 2번 상황만이 추가된 것인데, 2번 상황은 Base Color가 표시되지만 Emission은 적용되지 않는 상황이 된다. 
이 부분은 기본 흰색으로 나타나는데, 여기에 Color를 곱해서 Emission으로 보내면 끝임. ㅅㄱ