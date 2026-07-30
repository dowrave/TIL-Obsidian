
## 1. 블렌더
- `Shift + a`로 `cylinder` 생성
- 좌측 하단의 `Add Cylinder - CAp Fill Type : Nothing`으로 지정, 뚜껑을 없앰
- 도형 우클릭 - `Shade Smooth`
- 가로 엣지 생성 `Cylinder` 클릭 -  `ctrl + r`
	- 휠로 갯수 조절 가능. 3개 생성
	- 갯수 조절 후 클릭해서 위치도 조절 가능. 디폴트를 따라감.
- 새로 생성한 3개의 엣지 중, 가운데 엣지를 잡음 : `alt + 가운데 버텍스(엣지) 클릭`으로 해당 영역 지정 가능
- 상단의 `Proportional Editing` 활성화. 단축키는 `O`
- `S`키를 누른 채로 잡아 당겨서 선택된 버텍스/엣지를 밖으로 늘어나게 함.
- `A`키로 전체 버텍스를 잡고 `S + Z`를 눌러 Z축에 맞춘 스케일 축소

![[Pasted image 20241230134248.png]]
> 최종적으로 이런 모양이 나타남

- 모델을 클릭한 채로 `Export`함. 
	- `Limit to : Selected Objects` 체크
	- 확장자는 `FBX`

## 2. 유니티 VFX 그래프
- `Single Burst`
- `Output Particle Mesh`
	- 텍스처는 `Particle-System`의 그것 사용
- 검격의 이펙트 표현 자체는 위의 메쉬가 돌아가는 구조임
	- **텍스쳐는 메쉬 전체에 걸쳐 표현되지 않고, 메쉬의 일부 영역에만 나타남. 이게 돌아가면서 검격 이펙트를 나타내게 됨.**
	- `Initialize - Set Angle.XYZ`에서 초기 앵글 설정, `Update Particle - Add Angle.XYZ`에서 회전 각도 설정
	- 예제의 경우 업데이트는 `Sample Curve`에 `Age Over Lifetime[0, 1]`을 연결하고 `Multiply`를 통해 구현했음
![[Pasted image 20241230134629.png]]


## 3. 유니티 셰이더 그래프

### 초기 세팅
- `Blank Shader Graph`를 만듦.
- `Graph Settings`
	- `Active Targets : Visual Effect`
	- `Visual Effect : Material - Unlit`
- 디폴트 설정
	- `VoronoiScale : Float` 추가
	- `Voronoi` 노드 생성, `Cell Density`에 `Voronoi Scale` 연결
	- `Voronoi` 노드의 아웃풋을 `Fragment - Base Color`에 연결
- 저장 후 VFX 그래프의 셰이더 부분에 설정해서 결과를 봄

--> 보로노이 효과가 메쉬 전체에 적용된, 회전하는 메쉬가 나타남.

### 원형 그래디언트 구현하기
- 일단 더 진행하기에 앞서서, 강의에 나온 노드가 뭔지부터 적어보자
![[Pasted image 20241230140904.png]]

- `Polar Coordinate`
	- 직교 좌표계를 극 좌표계로 변환, `Out`은 `r, theta`의 극 좌표계 벡터가 됨
	- 오른쪽 Preview에서 
		- `r`이 중앙에서 멀어질수록 밝아지는 이유는 각 위치의 중심으로부터의 거리 값을 반영하기 때문이고
		- `theta`도 각도가 `0`일 때 `0`이고 `180`일 때 `1`이 되는 정규화가 된 것임

- 연결 구조
```
Split(r) - One Minus - Clamp - Power(2.xx) 

위의 아웃풋을 기존 Voronoi와 Multiply해서 Color에 연결
```
> 기존 `PolarCoordinate`의 `r`이 멀수록 밝아졌으니까, `One Minus`로 멀수록 어두워지도록 바꾼 다.
> `Power`는 해당 효과를 더 강화한 거고(제곱)
> `Clamp`는 `[0, 1]`을 벗어나는 값들은 제거해주기 위해 넣은 거임

이렇게 구현하면 검격이 돌아가는 중심 부분은 밝게, 먼 부분은 어둡게 나타난다. 어쨌든 메쉬는 모두 나타나고 있는 상태.

- 따라서 알파값도 추가해주는데, 이 과정에서 `Color`라는 프로퍼티를 추가한 다음
```
위의 Base Color에 연결한 Multiply 노드의 기존 아웃풋 연결을 해제하고
1. Multiply(with Color)
2. Multiply(with Color - Split - A)
으로 연결한 다음, 1의 아웃풋을 Base Color, 2의 아웃풋을 Alpha에 연결함
```

### 단조로운 효과 제거
 1. `Float : VoronoiSpeed` 라는 프로퍼티를 추가, `Time`과 곱해서 `Voronoi - Angle Offset`에 연결함
2. 추가로, `Voronoi`의 아웃풋에 `Power`를 연결, `Float : VoronoiPower`를 연결해서 `Dissolve`효과를 강화함


> 내 경우 추가로 메시 전체가 나타나는 현상을 줄이기 위해, 처음에 극 좌표계 구현에서 추가한 `Power`에 들어가는 부분을 `SlashTipsPower`라는 프로퍼티로 추가했음

## VFX 그래프에서 테스트
- 셰이더 그래프에서 오픈시킨 프로퍼티들을 변경해가면서 테스트하면 됨
- `HDR`로 색에 `Intensity`를 준 다음, `Voronoi Power`를 줄이면 색이 밝아짐
- 여기서 **`Voronoi Power`가 클수록 밝기가 감소**하는 쪽으로 영향을 준다는 것을 이용해서, **`Lifetime`에 따른 `Voronoi Power`의 조절**로 검격이 흐려지는 이펙트도 구현할 수 있다.

- 어쨌든 근본적으로 3D 메쉬의 회전을 이용해서 구현하는 것이기 때문에 정확히 어느 만큼의 이펙트를 내고 싶느냐는 사용처에 따라 전부 달라질 것임

![[Pasted image 20241230145056.png]]

### 어두운 부분 추가
- 지금까지 구현한 셰이더 그래프를 복붙
- `VoronoiScale, VoronoiSpeed, Size` 등을 더 작게 조절
- `Color`도 디폴트는 완전 검정으로 설정
	- `Alpha`를 1 이상으로 늘릴 수 있다. 더 늘릴수록 더 진해짐.

- 이렇게 구현하면 이펙트가 나오긴 하지만, 어두운 부분이 밝은 부분 위에 렌더링되게 됨
- VFX 그래프 파일의 인스펙터에서 `Output Render Order`을 조절하면 된다.
![[Pasted image 20241230145600.png]]
> 나중에 렌더링되는 게 화면 상에서는 위에 나타나므로, **메인인 `Bright`를 아래에 넣어줌.**


