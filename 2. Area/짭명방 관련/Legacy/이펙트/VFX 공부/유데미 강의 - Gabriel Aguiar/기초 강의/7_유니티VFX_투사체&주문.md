1. [[#강사님 스케치|강사님 스케치]]
2. [[#Projectile Mesh|Projectile Mesh]]
	1. [[#Projectile Mesh#블렌더 작업|블렌더 작업]]
	2. [[#Projectile Mesh#유니티|유니티]]
3. [[#텍스쳐 만들기|텍스쳐 만들기]]
4. [[#꼬리 궤적(Trail)|꼬리 궤적(Trail)]]
	1. [[#꼬리 궤적(Trail)#텍스쳐 만들기|텍스쳐 만들기]]
	2. [[#꼬리 궤적(Trail)#셰이더 그래프 작업|셰이더 그래프 작업]]
5. [[#Projectile Scene으로 변경|Projectile Scene으로 변경]]
6. [[#충격 효과 - Impact, Hit|충격 효과 - Impact, Hit]]
7. [[#발사 효과 - Muzzle|발사 효과 - Muzzle]]
	1. [[#발사 효과 - Muzzle#블렌더 : 메쉬 만들기|블렌더 : 메쉬 만들기]]
	2. [[#발사 효과 - Muzzle#유니티|유니티]]
8. [[#전기 버전 - ElectricLocal, ElectricWorld|전기 버전 - ElectricLocal, ElectricWorld]]
	1. [[#전기 버전 - ElectricLocal, ElectricWorld#Flipbook Animation 작성하기|Flipbook Animation 작성하기]]
9. [[#Electric Circle 만들기|Electric Circle 만들기]]
	1. [[#Electric Circle 만들기#메쉬 만들기 - 블렌더|메쉬 만들기 - 블렌더]]
	2. [[#Electric Circle 만들기#유니티|유니티]]
10. [[#전기 효과 - Impact, Muzzle|전기 효과 - Impact, Muzzle]]
	1. [[#전기 효과 - Impact, Muzzle#타격 효과|타격 효과]]
	2. [[#전기 효과 - Impact, Muzzle#Muzzle 효과|Muzzle 효과]]


- 여기서도 레퍼런스 참조와 스케치는 유용하다

## 강사님 스케치

1. `Projectile`
![[Pasted image 20250731150751.png]]
> 왼쪽부터 Trail, Particle, 3D Mesh, Texture

2. `Hit/Impact`
![[Pasted image 20250731150907.png]]

3. `Muzzle / Charging`
![[Pasted image 20250731151243.png]]
> 발사되는 부분의 이펙트. 


## Projectile Mesh
- 빈 오브젝트 아래에 파티클 시스템 오브젝트를 하나 만듦
- 1개의 파티클만 `Burst`되면 된다. 메쉬로 만들 거임.

### 블렌더 작업
- `UV Sphere` 생성. `Segments, Rings : 16, 16`
- `Numpad 1` 으로 **`Front View`** 설정. 
	- **텐키리스라면 `View - Viewpoint - Front`에 있음**
- `Numpad 5`로 **`Orthographic view`** 설정
	- 텐키리스라면 **`View - Perspective/Orthographic`**
- 아랫쪽에 있는 `Vertice`들을 지워준다. 
	- 강의에선 `B`를 누르고 드래그하라고 하는데
	- `Alt + Z`로 엑스레이 모드를 켜면 뒷쪽까지 보이므로 보이는 노드를 한꺼번에 지울 수 있음

![[Pasted image 20250731152334.png]]

`O`로 `Proportional Mode`를 켜고, 전체적인 구를 위로 잡아당긴다. `G + Z`가 무난해보임.

- `Object Mode`에서 우클릭 -> `Shade Smooth`으로 표면을 매끄럽게 처리
- 이름 `ProjectileMesh01`로 변경
- 이 상태에서 `Unwrap`하면
![[Pasted image 20250731153348.png]]
바로 이런 모양이 나옴. 

>[!info]
>UV Map이 저런 모양이므로 **텍스쳐를 만들 때는 중앙에서 바깥쪽으로 향하는 모양을 만들어준다는 생각을 하면 됨**

- `Export`하면 된다. 
- `Mesh` 체크랑 `Selected` 부분 체크.

> [!warning]
> 내보낼 때 `Object Mode`에서 해당 메쉬를 선택한 상태여야 함
> `Edit Mode`에서 a로 모든 노드를 잡아도 선택되지 않은 것으로 인식함


### 유니티
- 메쉬를 가져와 넣는데, 여기만의 특이한 설정으로는 아래 2개의 옵션을 모두 끈다.
	- `Material - Material Creation Mode`
	- `Animation - Import Animation`
- `Scale`도 100으로 설정

![[Pasted image 20250731154713.png]]
> `Crack01_Add`을 할당했을 때 이런 모양이 나온다. 

## 텍스쳐 만들기
- 이런 느낌의 텍스쳐를 만들면 된다.
![[Pasted image 20250731154922.png]]
- 대충 20 근처의 투명도를 가진 브러쉬에서 시작
> 이제 와서 깨달은건데 프로크리에이트에서도 브러쉬 투명도를 조절할 수 있었다. 화면에 슬라이드 바가 2개인데 왜 이걸 이제 알았지?
- ![[Impact02 (2).png]]

- `Impact02_Add`을 복붙해서 03으로 만들고, 텍스쳐 넣고, `Intensity`도 살짝 낮춰준다.
- `Particle System`의 `Material`을 새로 만든 머티리얼로 실험해보고, 만약 그래도 너무 밝다면 `Start Color`의 `Alpha`값을 낮춘다.

![[Pasted image 20250731162047.png]]
> 끝부분 처리가 좀 중요해보인다. 끝이 휘는 것보다는 직선으로 쭉 뻗는 방식이 이쁘게 나오는 듯.

- `Rotation over Lifetime`, `3D Start Size - Z축 1.1` 설정

- `ProjectieOut_AB`을 만들고 머티리얼도  `Alpha Blended`에 해당하는 걸 하나 만듦
	- 머티리얼의 `Intensity` = 0
	- `3D Start Size`는 0.9, 0.9, 1.5
	- 렌더링 순서 -1
	- 색은 똑같이 푸른 계통이되 `Value`만 `15 ~ 20` 정도로 설정한 값

- 추가 파티클 구현
	- 위치는 Z만 0.4로 설정
		- 도형의 꼭대기 부분이 +Z 방향이다. 그쪽에 조금 더 붙이는 모양새.
	- **`Simulation Space : World` 필수!!**
	- 이동 중에만 생성되므로 `Rate over Distance` 설정, 값은 3
	- 색은 2가지를 사용함 - 밝고 알파가 높은 파랑, 어둡고 알파가 낮은 파랑
		- 기존 `ProjectileOut`, `ProjectileOut_AB`의 색들 재탕해도 무방
	- 머티리얼은 `Beam01_Add` 사용

## 꼬리 궤적(Trail)
- `Effects - Trail` 오브젝트 추가
- 설정
	- `Time : 0.3`
	- `Width` 그래프 : 우하향(1 잠깐 유지 후)
	- `Color` : 밝은 하늘색 -> 어두운 하늘색. 알파는 100 -> 0
	- `Order in Layer` : -1

### 텍스쳐 만들기
![[Pasted image 20250731171812.png]]
- 먼저 일자로 소프트 브러쉬를 쭉 그린 다음 가운데를 지우고 디테일을 더하는 방식으로 작업
![[Trail01 (2).png]]
- 머티리얼은 `Impact03_Add`을 복붙해서 텍스쳐만 다시 할당
	- `Intensity = 1`로 설정
	- 강의에서는 저 곡면이 오른쪽으로 진행되어야 해서 텍스쳐를 좌우반전해서 다시 저장함
		- (진행 방향의 반대 방향으로 뾰족한 부분들이 향해야 함)
![[Pasted image 20250731174327.png]]
> 이런 느낌으로. 우 -> 좌로 이펙트를 움직였을 때의 `Trail`이다.


- 추가 작업
	- `Time`을 100으로 설정해서 결과물을 볼 수 있게 함
	- `Width`의 시작점을 2.0 ~ 2.5 정도로 설정
	- `Intensity`가 너무 클 필요는 없다. 주목도는 `Projectile`에 있어야 하지 `Trail`에 있을 필요는 없기 때문이다. 
	- `Dynamic Occlusion` : Off
	- `Cast Shadows` : Off
	- `Auto Destruct` : On
		- 활성화 시 `Projectile`의 움직임이 멈추면 자동으로 없어짐

### 셰이더 그래프 작업
- [[6_유니티VFX_범위효과#셰이더 그래프 Scroll & Distortion Shader]]에서 작업했음.
- `AddScroll`로 작성된 머티리얼이 있었다. 그거 복붙해서 텍스쳐만 다르게 할당해서 사용함.
- 셰이더 그래프의 타임 노드를 받는 애니메이션이 동작하지 않는 현상이 있었다. 여기서도 동일한 이슈가 발생해서 시각적으로 `Trail`이 움직이는 건 못 볼 듯?
- 대신 `Play`를 눌러서 게임 뷰에서만 관찰되도록 할 수는 있는 듯. 
	- 이 경우 `Projectile`의 이동을 멈추면 `AutoDestruct` 떄문에 파괴되는 현상도 있다. 테스트 때는 `AutoDestruct`을 끄고 해야 함.
	- 번거롭네~

- 머티리얼 설정값들 바꿔가면서 테스트해보셈~

- `ProjectileOut`에 설정되었던 `Impact03_Add`도 복붙해서 `AddScroll`로 바꿔서 진행함

## Projectile Scene으로 변경
- `Projectile` 프리팹에 `Rigid body`와 `Sphere Collider` 추가
	- `Rigidbody`에서 중력 영향은 제거
	- 가장 부모 오브젝트임
	- 파티클 띄우고 변경하려고 하는데 부모 오브젝트 클릭하면 이펙트가 안 보임;
	- 눈대중으로 `radius = 0.8` 정도에 `Center`의 `Z 포지션`은 `0.7`로 줬다.

- 강의에서 제공되는 `Projectile Move Script`가 있다. 이걸 할당하고 강의에서 제공되는 설명대로 설정.

![[Pasted image 20250731182043.png]]
> 지금까지 만든 파티클은 대략 이런 느낌이다. 

- 추가 작업
	- `Color` : 시작 부분에서 짤려보이는 게 이상함
		- 시작 지점에선 알파 0
		- 10% 지점에서 알파 100
		- **확실히 훨씬 낫다.**
	- `Trail`의 `Order In Layer`는 `-2`로 설정,
		- `Projectile`의 알파 블렌드 레이어보다 더 보이지 않게끔 설정함. 
	- `ProjectileMoveScript`에 `Trail`이 있다.
		- 파티클이 부딪혀서 사라질 때 `Trail`도 바로 사라지도록 설정됐는데, 이걸 서서히 사라지게 하는 옵션임

- 파티클이 보이지 않는 이슈가 있다. 
	- `Scene` 뷰에서 `Toggle Effect`를 켜면 됨. 반짝이는 표시가 있는 듯한 아이콘.

## 충격 효과 - Impact, Hit
- 새로운 오브젝트를 만든다. `vfx_Hit_v1`
	- `Beam` 오브젝트
		- `Beam` 머티리얼을 쓰며 `Projectile`과 같은 색을 사용
		- 커졌다 작아짐, 알파는 100 -> 0
	- `Particles`
		- `Projectile`의 `Particles` 복붙
		- `Burst` : 10 ~ 15로 변경
		- Looping 끄기
		- Simluation Space : Local로 변경
		- Start Speed : Two Curves로 변경
			- 처음엔 빠르다가 나중엔 느려지는 방식. 
			- 시작점은 5 ~ 20, 끝은 0 ~ 10
		- `Stretched Billboard`로 변경
	- `Impact`
		- `Beam` 오브젝트 복붙
		- `lifetime` 조금 더 길게
		- `Impact03_AddScroll` 머티리얼 적용
		- `Start Rotation` : -360 ~ 360
		- `Flip Rotation` 활성화
		- `Size` 5로 줄임
	- `Shockwave`
		- `Beam` 복붙으로 만듦
		- 머티리얼 - `Circle01_AddScroll`을 만든다.
		- `Beam` 대비 조금 더 오래 살고, 임팩트는 조금 더 흐려야 함
			- `Beam`이 0.2이므로 여기선 0.3 설정
		- `Shockwave`이므로 갈수록 커지는 이펙트
		- 페이드 아웃을 위해 끝 부분은 검은색

- 이렇게 만든 오브젝트를 `Projectile` 프리팹에 할당한 스크립트의 `Hit Prefab`에 붙인다.
- 그러고 테스트
![[Pasted image 20250731195404.png]]
- Trail 부분이 남아 있다. 그걸 제외한 나머지 부분은 `Hit`으로 인한 이펙트임.

- `Hit` 이펙트 추가 수정
	- `Size Over Lifetime`은 0보다 큰 값에서 시작
	- **이펙트가 경계면에 의해 잘리는 현상 수정**
		- 현재 씬에서 활성화된 카메라는 `MainCamera_BeautyShot`의 `VFX_Camera` 부분인데, 강의에선 `Culling Mask`가 `VFX`로 되어 있지만 내가 보는 상황에선 `Nothing`으로 나타남
		- 따라서 **`VFX` 레이어를 추가하고, `Culling Mask`에 `VFX`로 지정하고 `Hit` 이펙트만 `VFX` 레이어로 설정해주면 된다.**
	- `Impact`의 `Size Over Lifetime`의 피크 지점을 t = 0.7로 옮기고, 그 뒷부분은 0.5 까지 수축시키도록 한다.

![[Pasted image 20250731200023.png]]

>[!tips]
>- 여기서 잠깐 짚고 넘어가면, `MainCamera_BeautyShot`과 `VFX_Camera` 2개의 카메라가 씬에 있다.
>- 둘 다 카메라 컴포넌트를 갖고 있는데, `Output - Depth` 값이 큰 카메라가 보는 레이어들이 사용자 기준으로 가장 가깝게 보인다.
>- 이 상황에서 메인 카메라는 `Depth = -1`이고 `VFX_Camera`는 `1`이다. 메인 카메라는 VFX 레이어를 컬링마스크에 포함하지 않은 상황임.

## 발사 효과 - Muzzle

### 블렌더 : 메쉬 만들기
- `Cylinder`로 시작, 팔각기둥에 가까운 느낌을 만든다. 한쪽 면은 축소시키고 다른 쪽 밑면은 제거
- 피벗 옮기기
	- 마지막에 작은 밑면 부분을 클릭한 채로 `Shift + S -> Cursor To Selected` 
	- `Object Mode`로 이동, 우클릭 -> `Set origin - Origin to 3D Cursor` 설정
![[Pasted image 20250731201512.png]]
> `Unwrap`까지 하면 이런 모습
> 가운데로 모이는 버텍스들만 S키를 눌러서 더 줄여준다.

### 유니티
- 메쉬 설정 : 머티리얼, 애니메이션 옵션 끄고 스케일 팩터 100

![[Pasted image 20250731201931.png]]
머티리얼은 `Impact03_AddScroll`을 사용. 기초적인 파티클 시스템들을 세팅했을 때 나오는 결과다.

![[Pasted image 20250731202802.png]]
- 파티클 2개
- `3D Start Size` : `1, 1, 0.6` ~ `1.2, 1.2, 1`
- 0.6에서 시작해서 1로 커지는 구성
- 레이어 `VFX`로 변경

- 위에서 구현했던 `Hit` 내에 있는 `Beam`과 `Particle`을 `Muzzle`로 복붙
![[Pasted image 20250731203158.png]]
> 일시정지 한 상태에서 `Beam`의 위치 조절을 해줄 수 있다. 
> - 크기 조절도 함께 해주자.

- `Particle`도 기존엔 `Sphere`이었지만 이 경우 `Cone`이 적절해보인다. 
![[Pasted image 20250731203348.png]]
> 이런 느낌으루다가.
> 여기선 속도를 줄여서 저 정도로 튀진 않게 구현하면 됨

나머지는 `Muzzle` 프리팹 저장하고, `Projectile`의 `Muzzle`에 할당하고 테스트.
> 여전히 `Projectile`의 파티클들만 보이지 않는 게 거슬린다. `Projectile`의 나머지 파트는 운동 갔다 와서 마저 함. 
> **오늘 다 끝내려고 했는데 실습까지 겸하면 무리다. 6시간 강의라고 내가 배우는 데 6시간이 걸린다는 뜻은 아님 ㅋㅋ;**

## 전기 버전 - ElectricLocal, ElectricWorld
- 기본적으로 노랑 - 주황 계열의 색으로 바꾼다.
- 특이사항

1. `Particles`의 색
![[Pasted image 20250731220049.png]]

2. `Trail`의 색
![[Pasted image 20250731220227.png]]
> 노랑으로 바뀐 다음 어두워진다는 느낌임

이렇게 하고 기존의 `Projectile`을 할당했던 스크립트에 `Electric` 버전을 할당해보면 됨

### Flipbook Animation 작성하기
- 강의에선 9장의 이미지를 그렸음. 포토샵을 이용하므로 선을 그리고 그걸 따라 Path를 만들고 Outer glow를 적용하는 느낌. 
- 전기 선을 그리는 거니까 너무 연속적인 애니메이션일 필요도 없다. 강의에서도 애니메이션 돌려봐도 전기가 찌리릿한다는 느낌은 아님.
- 프로크리에이트로도 가능은 할 거임. 대충 3개의 이미지를 그린 다음에 픽셀 유동화 기능을 이용해도 될 것 같은데?

>[!info]
> - 프로크리에이트로 작업했음.
> - 애니메이션의 개별 장면을 원본 + 가우시안 블러로 뽀샤시한 걸 적용. 각 장면을 그룹으로 묶음
> - 애니메이션에서는 개별 레이어가 1개의 그림인 것으로 보이며, 아래에서 위로 서서히 진행되는 듯 하다.
> -  `레이어 공유 - PNG 파일`로 각 그룹을 PNG 파일로 뺄 수 있음.
> - 강의에서도 나오는 팁으로, **키 프레임을 먼저 그리고 그 사이사이를 메운다**는 느낌으로 그려나가면 된다. 이게 그 동화인가 뭔가 하는 그거냐?


![[Electricity01-1 (2).png]]

![[Electricity01-2 (2).png]]

![[Electricity01-3 (2).png]]

![[Electricity01-4 (2).png]]

![[Electricity01-5 (2).png]]

![[Electricity01-6 (2).png]]
이런 느낌으로 작업했다. 너무 꼬불꼬불할 필요는 없을 것 같기는 함.

- 이 파일들을 어떻게 스프라이트 시트로 묶는가?
- `GlueIT`이라는 소프트웨어가 있다. 지금도 깃허브에서 오픈되어 있는 프로젝트인데, 여기서 다운 받음.
![[Pasted image 20250731224620.png]]
> 모든 프레임을 넣고, 몇 개의 줄로 된 스프라이트 시트를 만들 것인지를 선택하고, `Save`를 누르면 된다. 만들어진 이미지 이름은 `Electricity01-2x3.png`

![[Electricity01-2x3 (2).png]]

- 기존 머티리얼 중 `Impact03_Add`을 복붙. `Electric01_Add_2x3`으로 명명.
- 오브젝트 중 `Particles`을 복붙, 이름은 `ElectricLocal`
	- `ParticleSystem`에서 `Texture Sheet Animation`이라는 부분이 있다.
		- 이 중 `Tile` 옵션에는 해당 텍스쳐 시트 애니메이션이 몇 * 몇인지 판단해야 하는 부분이 있음. **머티리얼 이름이나 텍스쳐 이름에 그리드가 어떻게 되는지 명명하는 게 그래서 좋다는 것이다.** 
		- `Tile`의 경우 `X, Y` 옵션으로 주어진다. **X가 열의 갯수, Y가 행의 갯수다.**
	- 노이즈 제거하고 파티클 크기 키우고 머티리얼도 텍스쳐에 스프라이트 시트 적용해서 할당하면 아래처럼 나타남.
![[Pasted image 20250731225823.png]]
- `Texture Sheet Animation`의 옵션 중에는 `Frame Over Time`이라는 게 있다.
	- 기본적으로 **`Curve`로 지정되는데, 이 경우 각 파티클은 자신의 애니메이션을 재생**함.
	- **`Random Between Two Constant` 지정시 스프라이트 시트의 특정 부분만을 재생**한다 - 즉 애니메이션 효과가 나타나지 않지만, 스프라이트 시트에 있는 다양한 형태의 파티클 중 하나만 재생하는 식임

- 추가 옵션 설정
	- 파랑을 주황으로 변경, 알파값도 더 키움
	- `Simulation SPace`는 `Local`로 해서 `Projectile`과 함께 이동하도록 함
	- `Blink` 효과 추가 
		- `Color over Lifetime` 
![[Pasted image 20250731230846.png]]
	-  `Size Over Lifetime`
		- 서서히 작아지되 0.5 정도에서 끝남
	- `Rotation Over Lifetime`
		- -90 ~ 90

![[Pasted image 20250731231139.png]]
> 현재까지의 결과물.


- 이제 투사체가 이동할 때 남기는 `Electric Particles`의 잔상을 만든다.
- `ElectricLocal`을 복붙, `ElectricWorld`를 만듦.
	- `SimulationSpace : World`
	- `Rate Over Distance : 3`

## Electric Circle 만들기
- 위에서 `Flipbook Animation`으로 `Electric` 파티클들을 만들었잖음?
- 그거의 원형 버전이다. 
![[ElectricCircle01-2x3 (2).png]]
- 스프라이트 시트를 만든다. 이건 내가 만든거.
- 강의에서도 머티리얼을 뭘 만들고 할당하라는 말은 없음

### 메쉬 만들기 - 블렌더
- 모든 각도에서 이펙트를 보기 위해 메쉬도 추가로 구현한다.

- `Mesh - Circle`을 만듦
	- `Vertices` : 16
	- `Extrude`로 Z축에 대해 0.3만큼 뽑아냄
	- UV 맵 생성, 가운데의 넓은 원을 줄여준다. 버텍스들을 잡고 `S`키로 조절하면 됨.

![[Pasted image 20250801140340.png]]

- 상단 `UV` 메뉴에서 `Live Unwrap`을 켠다.
- `Ctrl + R`로 도형의 중앙에 버텍스들을 추가한다. 
	- `Ctrl + R`로 클릭한 상태에서 `Cut` 값을 `3`으로 지정

![[Pasted image 20250801140856.png]]

- 가장 위와 아래의 버텍스들 잡고 가운데로 모이는 모양을 만든다.
- 가장 가운데의 버텍스들을 잡고 밖으로 살짝 크게 만든다.
![[Pasted image 20250801141251.png]]

>[!tips]
>UV Map에서 `Image - Open`으로 텍스쳐 이미지를 할당해볼 수 있다.

![[Pasted image 20250801141356.png]]
> 만든 텍스쳐가 메쉬에 어떻게 들어가는지 볼 수 있음.

- 이렇게 작업하면 텍스쳐가 메쉬에 들어가기 때문에 어떤 각도에서든 이펙트를 볼 수 있게 됨
> 물론 `Billboard`로 작업해도 항상 카메라를 향하니까 이펙트를 볼 수 있다는 건 똑같지만, 앵글에 따른 이펙트의 변화가 있으려면 메쉬에 텍스쳐를 넣는 식의 구현이 좋다는 의미인 것 같다. 

- `Ring01`이라는 이름으로 fbx 파일과 blender 파일을 저장한다.
	- 오브젝트 모드에서 메쉬를 선택한 상태로 `Selected` 및 `Mesh` 체크는 잊지 말 것.
### 유니티
- 메쉬는 기존에 해왔던 것과 동일하게 `Scale : 100`, 머티리얼과 애니메이션은 끈다.
- `ProjectileOut`을 복붙해서 설정함.
	- `Duration` : `1.00`
	- `Looping : True`
	- `Start Lifetime : .6 ~ .8`
	- `3D start Size : 1, 1, 1, ~ 1.25, 1.25, 1.5`
	- `Rate over Time : 5 ~ 7` / Burst 해제
	- `Texture Sheet Animation` : `Grid, x=2, y=3`
	- 색은 `ElectricLocal`에서 가져옴
	- 머티리얼은 기존의 텍스쳐 시트 애니메이션에 사용했던 머티리얼에서 텍스쳐만 바꿔서 사용함


- `Velocity Over Lifetime` 설정
	- `0, 0, 0` ~ `0, 0, -2.5`
	- 파티클이 뒤로 나가는 듯한 이펙트. `Projectile`과 유사한 효과가 된다. 

![[Pasted image 20250801142820.png]]
> 이번에 구현한 이펙트만 놓고 보면 이런 모양이고

![[Pasted image 20250801142850.png]]
> 전체적으로 보면 이런 모양이다. `ProjectileOut`은 눈으로 보면서 어떤 위치로 설정하면 좋을지 직접 Z축의 `Transform`을 만지면서 테스트해보자.


- `Color over Lifetime` : `ElectricLocal`에서 가져온다. 
	- 전기 효과니까 깜빡이는 효과를 `ElectricLocal`에서 구현한 적이 있었다. 그걸 사용하는 것.
- `Rotation over Lifetime` : `-720 ~ 720`


## 전기 효과 - Impact, Muzzle

### 타격 효과
- `vfx_Hit_Electric_v1`을 만든다. 
	- 파티클 중 하나만 노란색으로 만듦
	- 여기선 기존의 `Impact, Circle` 대신 `Projectile`에서 만든 `ElectricLocal, ElectricCircle`을 복붙한다. `Transform`도 리셋해주자.
	- 자세한 설정은 강의를 직접 보자.
		- `ElectricCircle`의 경우 
			- `Projectile`에선 투사체를 감싸는 모양이므로 `Local`이지만
			- `Hit`에선 사용자에게 이펙트가 직접 보여야 하므로 `View`로 구현한다.

![[Pasted image 20250801144311.png]]

### Muzzle 효과
![[Pasted image 20250801144741.png]]

