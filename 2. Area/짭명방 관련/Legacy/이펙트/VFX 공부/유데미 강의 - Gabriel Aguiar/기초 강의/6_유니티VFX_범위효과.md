
1. [[#레퍼런스 참고하기|레퍼런스 참고하기]]
2. [[#스케치Sketching|스케치Sketching]]
3. [[#이펙트 만들기|이펙트 만들기]]
	1. [[#이펙트 만들기#Anticipation|Anticipation]]
	2. [[#이펙트 만들기#Climax|Climax]]
4. [[#원기둥 이펙트를 위한 메쉬 만들기|원기둥 이펙트를 위한 메쉬 만들기]]
	1. [[#원기둥 이펙트를 위한 메쉬 만들기#UV Map 만들기|UV Map 만들기]]
	2. [[#원기둥 이펙트를 위한 메쉬 만들기#유니티|유니티]]
5. [[#텍스쳐 : Impact 01|텍스쳐 : Impact 01]]
	1. [[#텍스쳐 : Impact 01#추가 효과 적용|추가 효과 적용]]
6. [[#텍스쳐 : Impact 02|텍스쳐 : Impact 02]]
	1. [[#텍스쳐 : Impact 02#특기할 점 - 파티클 갯수와 크기/회전 조정|특기할 점 - 파티클 갯수와 크기/회전 조정]]
7. [[#충격파|충격파]]
8. [[#바닥 균열|바닥 균열]]
9. [[#알파 블렌드 쉐이더|알파 블렌드 쉐이더]]
10. [[#불 이펙트 버전|불 이펙트 버전]]
11. [[#Nature 버전 - 파트 1|Nature 버전 - 파트 1]]
12. [[#Nature 버전 - 파트 2|Nature 버전 - 파트 2]]
	1. [[#Nature 버전 - 파트 2#블렌더|블렌더]]
	2. [[#Nature 버전 - 파트 2#텍스쳐 만들기|텍스쳐 만들기]]
	3. [[#Nature 버전 - 파트 2#유니티|유니티]]
13. [[#셰이더 그래프 : Scroll & Distortion Shader|셰이더 그래프 : Scroll & Distortion Shader]]
	1. [[#셰이더 그래프 : Scroll & Distortion Shader#스크롤 구현|스크롤 구현]]
	2. [[#셰이더 그래프 : Scroll & Distortion Shader#Distortion 구현|Distortion 구현]]
14. [[#최종 수정|최종 수정]]

## 레퍼런스 참고하기
- 핀터레스트, 아트스테이션, 유튜브 등에 좋은 레퍼런스들이 많이 있다.
- 이들은 공부, 학습용으로만 사용하자.

## 스케치Sketching
- [[5_유니티VFX_스파크만들기]]에서 구현했던 스파크는 기초적인 이펙트라서 스케치가 필요한 작업은 아니었다. **더 복잡한 이펙트를 구현하고자 한다면, 레퍼런스와 함께 스케치 작업**도 하는 게 좋다.

- 브러쉬는 너무 진한 걸 쓸 필요는 없다. 희미하게 연필로 스케치한다는 느낌으로 사용하면 좋음. 강의에선 포토샵을 이용했으며, 브러쉬에 `Opacity`를 주는 방식으로 작업함
	- 작업 후에 너무 흐릿하다는 인상이라면 단순히 레이어를 복붙한 다음 합치기만 해도 진해진다.

- 여기서 그려지는 내용은
1. 예기, 전조`Anticipation`
	- 원기둥이 작아짐
	- 원기둥의 옆면에는 텍스쳐 스크롤링이 일어남
	- 바닥에도 원기둥의 밑면보다 큰 원 모양이 있음
	- 위 요소들은 모두 시간에 따라 서서히 작앚미

2. 폭발 - `Climax`
	- 말 그대로 폭발.
	- 파티클이 사방으로 튀는 효과.

3. 흩어지기 - `Dissipation`
	- 폭발이 일어난 곳에 흔적이 남음

강의에선 이런 느낌으로 진행됨
![[Pasted image 20250728130439.png]]

- 위의 내용은 기본적으로 추상적인 부분임. 속성을 추가하고자 한다면 그에 맞춘 속성 요소들을 추가하는 게 좋다. 
![[Pasted image 20250728130509.png]]

- **스케치의 유용성**
	- 레퍼런스와 스케치를 볼 때, 이들을 분해해서 볼 줄 알아야 함
![[Pasted image 20250728130901.png]]
- 무엇이 3D 메쉬이고 텍스쳐이고 파티클인지 구분할 줄 알아야 한다는 의미.

## 이펙트 만들기
- 실습을 따라가는 걸로 함. 특기할 내용이 있는 경우에만 적어둔다.

### Anticipation

- `Renderer - Max Particle Size` 설정 관련
	- 강의에선 3으로 설정함
	- 이런 의문이 들었다 : 파티클 하나가 뷰포트에서 최대로 차지할 수 있는 크기`Size`라는 의미인데, **1을 넘는 값이 의미가 있는가?**
	- **의미가 있다.** 예를 들어서 엄청 큰 이펙트가 있고, 이 이펙트의 세부적인 효과만 카메라에 나타나는 상황도 얼마든지 있을 수 있다. 그래서 1을 넘는 값도 의미가 있음. 

- `Renderer - Render Mode`
	- `Billboard` : **파티클이 항상 카메라를 바라보도록 회전한다. 2D 평면이 카메라에 직각으로 놓인 것처럼 보인다.**
		- 일반적인 불꽃, 연기, 먼지 효과 등
	- `Stretched Billboard` : **파티클이 카메라를 보되, 이동 방향으로 늘어난다. 파티클의 속도에 비례해 늘어나는 정도를 조절할 수 있다.**
		- **속도감, 움직임의 흐름을 강조**할 때 쓸 수 있다.
		- 비, 눈, 총알 궤적, 워프 효과, 캐릭터의 이동 스피드 라인 등등
	- `Horizontal Billboard` : **파티클의 평면이 항상 월드 공간의 `XZ` 평면에 평행하게 유지된다. 카메라가 어디에 있든 파티클은 수평으로 놓인 텍스처처럼 보인다.** 
		- 땅에 퍼지는 안개, 발 밑의 마법 원, 물 위의 나뭇잎 등등
	- `Vertical Billboard` : 월드 공간의 Y축을 중심으로만 회전한다. 파티클의 평면은 항상 XZ 평면에 수직이다.
		- 수직으로 솟아오르는 연기 기둥, 폭포 물줄기, 오로라/빛 기둥 등


- `Fade In` 효과
	- 0에서 시작, 약 90% 지점에서 최대 알파값을 가진 다음, 다시 100%에 0으로 설정.

- 크기가 서서히 감소하는 효과
	- `lifetime`이 끝나더라도 `Size`를 0으로 하지 않고 `0.1` 정도로 남겨둔다. 

- **여러 파티클 시스템을 혼합할 때, 복붙을 애용하자.** 

-  싱크 맞추기
	- `BeamGround` : 바닥의 플레어 효과가 서서히 작아짐
		- `Start Lifetime` : `1`
	- `ParticlesIn` : 파티클들이 가운데에 모이는 효과
		- `duration` : `0.75`
		- `start lifetime : 0.2 ~ 0.25`
	- `BeamGround`의 라이프타임과 `ParticlesIn`의 파티클이 완전히 사라지는 시점을 맞춤

- `ParticlesIn` : 파티클 모이는 효과 구현
![[Pasted image 20250728144138.png]]
- 원과 주위 파티클들이 함께 가운데로 모이는 모양임
	- **`HemiSphere`과 `-` 속도로 구현 가능.**
- `Shape : Hemisphere`
- `StartSpeed : Curve, 0에서 시작해서 -15`
	- 이러면 파티클들이 가운데로 모이는 효과가 난다.
	- `Random Between Two Curve`를 이용해 2개의 곡선 사이의 임의의 값이 찍히게 하는 것도 좋다. **랜덤성은 항상 좋다.** 
- `Render Mode` : `Stretched Billboard`
	- `Speed Scale` : `0.05`
	- **`Speed Scale`이 없으면 `Stretched Billboard`에서는 나타나지 않음.**
- 파티클은 시간에 따라 작아져도 되지만 커져도 효과 자체가 크게 달라보이진 않는다.
	- 그래도 큰 사이즈에서 작아지는 게 더 이쁜듯

- 색깔 관리
	- 파티클의 경우 `Random Between Two Colors`을 사용해주면 좋다. 
![[Pasted image 20250728145429.png]]
- 이런 느낌으로 `Opacity`도 다르게 가져가고 색상도 다르게 해주는 느낌.
- 그리고 둘 중 메인 색깔을 `BeamGround`(바닥)에 붙여넣은 다음, 세부 조정은 거기서 진행하면 됨.
	- **일일이 색을 다시 찾아다닐 필요는 없다. 저 값들은 다 복사가 되는 값들이기 때문임.**


### Climax
- 위에서 작성한 `BeamGround + ParticlesIn`을 **복붙**해서 시작.

- `Anticipation`과의 딜레이
	- `Anticipation` 파트가 1초였기 때문에 `Start Delay`를 기본 1초로 설정한다. 
	- 하지만 **모이자마자 터지는 것보다는 아무 것도 없는 약간의 유예가 더 임팩트가 있다.** 그래서 `1.2`초로 설정.

- **색깔 설정**
	- `Anticipation`에서는 투명 -> 진해지는 효과였지만, `Climax`에서는 반대로 진함 -> 투명으로 진행한다.
	- 흰색 -> 검은 투명으로 진행

- 크기 설정
	- 클라이맥스는 0.8 -> 1 로 진행되게끔 설정

이 2개의 설정으로 쾅! 하면서 이미 큰 이펙트가 나타나고, 이후 서서히 커지면서 사라지는 효과가 나타난다.

- `ParticlesOut` 설정
	- 거의 비슷하다. 딜레이도 1.2초로 맞추고..
	- `StartLifetime : 0.4 ~ 0.9`
	- `StartSpeed : 2 ~ 15`
	- `Emission` : `Bursts, 20 ~ 25`
	- 색깔 : 불투명도 1인 흰색 -> 불투명도 0인 검은색
	- `Shape` : `Circle`로 변경, 반지름은 크기 보고 결정
		- `Hemisphere` -> `Circle`로 바꾸면 `xz`평면에 붙어서만 파티클이 튀어나간다. 

- `Anticipation` + `Climax` 테스트 
	- **단순하게 두 오브젝트를 동시에 잡고 실행**시켜보면 됨. `Climax`에 넣은 오브젝트들은 1.2초 딜레이를 설정했으니 자연스럽게 실행됨.

- 더 강한 임팩트 주기
	- `BeamGround + ParticlesOut` 복붙, `ParticlesOut` 삭제, `BeamGround -> Beam`으로 설정
	- `Billboard`로 변경
	- `Start Lifetime` : 0.2초
	- `Start Size` : 5
	- `Size over Lifetime` : 오른쪽으로 하향
	- `ParticlesOut`을 다시 `HemiSphere`로 변경

![[Pasted image 20250728151314.png]]
> 최종적으로 이런 느낌이 됨. 저 3개의 파티클 이펙트를 잡아서 실행시키면 된다. 
> 마지막에 구현한 `Beam`은 터지는 순간에 잠깐 이미지를 보여주는 것 뿐임


## 원기둥 이펙트를 위한 메쉬 만들기
- 작업은 블렌더로 진행. 강의랑 버전이 많이 다르기 때문에 기능들은 적어가면서 함
- 이 작업은 `4.3.2`버전 기준.

- `Shift + A`로 `Cylinder` 메쉬 생성
	- `Vertices` : 16
- 원기둥의 위아래 `Face` 제거

- **원기둥의 밑면을 피벗으로 잡기**
	- 원기둥의 아랫 버텍스들을 잡음(점 하나 클릭하고 `Alt + 클릭`)
	- 3D 커서를 원기둥 아래의 중심으로 위치시키기 : `Shift + S` -> 
	`Cursor to Selected`
	- `Object Mode`로 돌아온 다음 선택된 상태에서 우클릭 -> `Set Origin -> Origin To 3D Cursor`

> 피벗을 옮기는 과정은 나중에 유니티로 갔을 때 빅-도움이 된다.

- 다시 `Edit Mode`에서 윗면의 버텍스들을 잡고 `G + Z` 클릭, 숫자 `2`클릭해서 길이를 늘여준다.
- `Cylinder`의 이름을 `Cylinder_NoTop`으로 바꿈

### UV Map 만들기

- 이렇게 Edge들을 잡아야 함
![[Pasted image 20250728154314.png]]
1. `Alt + 밑면의 Edge 중 하나 클릭`
2. `Shift + 옆면의 한 Edge 클릭`
3. `Alt + Shift + 윗면의 Edge 중 하나 클릭`

- 이렇게 잡은 상태에서 `Ctrl + E` -> `Mark Seam`
	- 저 `Edge`들이 UV Unwrap할 때 잘리는 지점들이 된다. 

- 모든 메쉬 지정 -> `U` -> `Unwrap` 
- `UV Editor` 창을 열면
![[Pasted image 20250728155133.png]]
이런 느낌의 창이 나타난다. 
1. `UV Map`의 좌측 상단에 빨갛게 동그라미 쳐둔 부분은 메쉬가 클릭되지 않더라도 `UV Map`이 계속 나타나게 하는 동작이다.
2. 강의 내용대로 따라간다면, 원기둥의 윗면 버텍스들이 UV Map에서도 y축으로 윗쪽에 위치해야 한다. 이를 위해서는
	- `R` 클릭 후 숫자 클릭하면 정해진 앵글로 돌릴 수 있음
	- `UV Map`을 가득 차게끔 구현한다.

![[Pasted image 20250728155354.png]]

- 여기까지 작업했으면 블렌더 파일과 메쉬를 저장한다.
- `Ctrl + s` : 블렌더 파일 저장
- 내보내기 : 메쉬를 선택한 상태로 진행
	- `Export` -> `FBX`
	- 설정
		- `Limit to : Selected Objects`
		- `Object Types : Mesh`만 선택

---
### 유니티
- 모델을 옮김. `Scale Factor`만 10으로 설정하고 `Apply`

 > [!warning]
 > 여기서 모델이 정상적으로 나타나지 않는 이슈가 있었다.
 > - **메쉬 크기 & 머티리얼이 혼합돼서 발생한 이슈**로 보임
 > - 머티리얼의 경우 기존에 구현했던 `ParticlesAdditive_HDR`로 적용하면 잘 보이지 않는다. 
 > - **`Export` 자체는 강의에서 말한 대로 적용하면 될 듯. `Selected`만 + `Mesh`만**
 
- `Renderer` 설정
	- `Render Mode` : `Mesh`
	- `Render Alignment` : `View -> Local`
		- `View`로 둘 경우 항상 카메라를 향함
		- `World` 설정도 있는데 이는 자체적인 `transform`이 적용되지 않아서 `rotation` 등이 반영되지 않는다.
- `Transform` : `-90, 0, 0`

## 텍스쳐 : Impact 01
- 이미지 프로그램에서 작업. `1000 * 1000 캔버스`
![[Pasted image 20250728173451.png]]
이런 이미지를 만들면 됨. 작업 방향은 큰 것부터 시작해서 작은 것들을 다듬어나가는 방식.
1. 소프트 브러쉬로 아랫면을 그음
2. `Smudge`로 어두운 쪽에서 밝은 쪽으로 스파이크를 만듦
3. `Smudge`를 더 작게 해서 각 스파이크에 삐쭉삐쭉한 느낌 추가
4. 반대로 밝은 쪽에서 어두운 쪽으로도 스파이크를 더 만듦
5. 지우개를 이용해 흰색과 어두운 색의 중간 지점들을 추가

![[Tut_Impact01-1 (2).png]]
> 프로크리에이트에서 작업한 내 결과물. 뭔가 느낌이 다르지만 완성도에 집착하진 말자

- 텍스쳐 파일에는 `1024 x 1024` 그대로를 넣는다.
- 기존 `Beam01_Add`을 복사해서 텍스쳐만 지금 만든 걸 넣고 메쉬를 띄워보면
![[Pasted image 20250728180805.png]]

이런 느낌의 결과물이 나옴. 
> 확실히 끝이 뾰족하냐 뭉툭하냐는 차이가 있다..

- `3D Start Size`를 만져서 이펙트의 크기를 만져줌
	- `X, Y`에 비해 `Z`축을 크게 죽이면 오오라 같은 이펙트가 되기도 한다. 참고해두자.
![[Pasted image 20250728181103.png]]

- `Size Over Lifetime`
	- `X, Y`는 위로 볼록한 감소, `Z`는 일정하게 1로 유지

- **`Rotation Over Lifetime`**
	- 처음 써보니까 볼드 표시. `Lifetime` 동안 `Angular Velocity` 만큼 돈다.

- `Cylinder`를 `Climax`에도 적용함
	- 딜레이 적용
	- 짧은 Lifetime
	- `X, Y`는 그대로, `Z`축만 서서히 커짐.
	- 색깔 : 흰색 -> 불투명도 감소 검은색
		- **페이드아웃 효과를 줄 때 검은색이 확실히 더 사라지는 듯한 효과가 남**

### 추가 효과 적용
- **`Vertical Cylinder`가 사라진 후에 수직으로 올라가는 잔해 파티클 효과 추가**
	- 있는 게 훨씬 이쁘다. 
- `ParticlesVertical` 오브젝트를 `ParticlesOut`을 복사해서 생성
	- 더 오래 남아 있고, 더 느리게 움직이며, 더 작다.
	- `Shape` : `Cone`
		- `Angle` : `0`으로 하면 원기둥이 됨.
		- `Radius`도 늘려준다
		- **`Radius Thickness`**
![[Pasted image 20250728183006.png]]
> `Radius = 1, Radius Thickness = 0.2`인 옵션이다.
> 여기서 수직으로 올라가는 파티클들이 생성되는 부분은 바깥 원과 안쪽 원 사이인데, 저 사이의 간격이 `Radius Thickness` 옵션이다.
> 만약 `Radius Thickness = 1`이라면 원 전체에서 파티클이 생성될 수 있다.


## 텍스쳐 : Impact 02
- 똑같이 `1024 x 1024`를 만듦
![[Pasted image 20250728183422.png]]
> 이런 이미지를 만들면 된다. 

1. 일단 전체적인 모양부터 잡음 : 디테일을 그리기에 앞서서 중앙을 잡아야 함
2. 작업 과정은 똑같다 : 가운데에서 바깥쪽으로 튀어나가는 굵은 선부터 작업하고, 가는 선으로 작업하면 됨 
3. 지우개로 지워서 모양을 다듬음

![[Tut_Impact02 (2).png]]
>만질수록 모양이 마음에 안 들어지는 이상한 현상..

![[Pasted image 20250728194205.png]]
- `Beam`을 복붙해서 `Impact`라는 걸 만들고, 기존의 `Additive` 머티리얼에 텍스쳐만 변경해서 하나 생성한 다음 할당해주면 된다
- 1.22초 시점의 이미지인데, 만든 이미지와 비교했을 때 아래가 잘린 것처럼 보임. **이건 `Renderer - Pivot`에서 `Y`값을 만져줘서 수정할 수 있음.**

### 특기할 점 - 파티클 갯수와 크기/회전 조정
- `Emission`에서 파티클 갯수는 1개가 아니라 4~5개로 설정한다.
![[Pasted image 20250728194417.png]]
> `Additive`이므로 이렇게 보임. 

- 여기에 추가로 `3D Start Size`, `3D Start Rotation`을 랜덤하게 지정하면 더 다채로운 효과가 나옴.
- `Size` : `(3, 4, 1) ~ (4, 6, 1)`
- `Rotation` : `(0, 0, 0) ~ (0, 360, 0)`

![[Pasted image 20250728194947.png]]
> 이 효과를 `Climax`에 붙여주면 됨. 

- 결과적으로 **2D `Billboard`이미지를 붙였지만 3D 처럼 구현이 되고 있는 걸 볼 수 있다!** 
	- 단순히 여러 개의 파티클을 `3D Size`와 `3D Rotation`만을 바꿨는데 말이다.

![[Pasted image 20250728195216.png]]
> `Climax`가 시작하는 지점의 효과. 


## 충격파
- 이미지 프로그램에서 `Circle01`이라는 `1024 x 1024` 이미지를 만듦
- ![[Pasted image 20250728195532.png]]
![[Tut_Circle (2).png]]
- 구현은 동일하다.
	- `Beam`을 복붙해서 구현, 머티리얼도 `Additive` 복붙해서 텍스쳐만 변경
	- `Size Over Lifetime`은 위로 볼록한 증가하는 그래프
	- `Start Size`는 13
	- `Lifetime`의 경우
		- 기존의 0.2초도 괜찮지만
		- 값이 클수록 퍼지는 속도가 느리고, 효과가 묵직해보인다.
	- `Beam`과 동일하게 바닥에 붙었기 때문에 `Horizontal Billboard`

![[Pasted image 20250728200454.png]]
> 여기서 바닥면에 붙은 가장 바깥원이 이번에 구현한 `Shockwave`

## 바닥 균열
- 이미지 프로그램 - `Crack01`

1. 투명도를 매우 낮춘 브러쉬, 굵기는 어느 정도 굵게. 가운데에서 뻗어나가는 9개의 부드러운 지그재그 선을 그림

>[!note]
>- 요점 : 어떤 텍스쳐를 그릴 때는 **선 하나를 그리더라도 불투명도가 구분되는 지점들이 있다면 좋다는 거임**
>- 그래서 굵은 선에서 시작해서 가는 선들로 디테일을 추가하는 방식으로 진행된다.
>- **만약 어떻게 그려야 할지 모르겠다? : 항상 레퍼런스를 참고하시오**



 ![[Tut_Crack (2).png]]
 > 뭔가 Crack이라기보다는 수도권 교통망 같음ㅋㅋㅋㅋㅋㅋㅋ
 
 ![[Pasted image 20250728203259.png]]
 > 강의에선 이런 모양임.
 
이후엔 머티리얼 만들고 `Shockwave`를 복사해서 만든다
- `Dissipation`의 일부로 사용할 이펙트이므로 
	- `Lifetime`은 다른 것들보다 더 길게 설정
	- 만약 어둡다면 `Start Color`의 명도를 높여도 된다.
![[Pasted image 20250728204108.png]]
이런 느낌으로 나온다.

> 강의랑 비교했을 때, **확실히 선이 굵직한 게 더 멋있어보인다.** 

## 알파 블렌드 쉐이더
- **`Additive` 쉐이더는 어두운 색을 만들지 못함**
- 기존에 `Additive` 쉐이더를 만들었으니까 단순히 복붙해서 설정만 바꾸면 만들 수 있다. 
	- `Blending Mode`의 `Additive` -> `Alpha`로 변경
	- 이름은 `ParticlesAB(AlphaBlended)_HDR`

- `CrackAB` 생성 : 기존의 `Crack`을 복붙해서 머티리얼을 새로 적용함
	- `Start Color`는 기존의 핑크/보라에서 검정색, 알파도 100으로 설정한다
	- 그 외) `Burst Count : 2`, `Start Rotation : 0`

- 추가로 기존 `Crack` 오브젝트의 **`Sub Emitters` 옵션**을 켠다.
	- `Birth`에 새로 만든 `CrackAB`을 지정한다.
		- `Birth` 이외에도 아래의 옵션이 있음 
			- `Collision`
			- `Death`
			- `Trigger`
			- `Manual`
		- 지금은 `Crack`과 함께 `CrackAB`을 쓰고 싶은 거니까 `Birth`로 설정
	- `Inherit`
		- **자신의 속성을 `Sub Emitter`에 전달할 수 있다.** 색깔, 크기, 회전, 수명, 지속시간 등등
		- 여기선 **`Size, Rotation`을 설정.**

- `Crack AB` 추가 설정
	- `Start Delay` : 0
		- `Crack`의 `Start Delay`를 따라가면 된다.
	- `Start Size` : 1
	- `Start Lifetime` : 3
		- 여기선 상속받지 않았으니까 `Crack`이 발생하는 시점에 3초의 생명 시간을 가진 이펙트가 생성된다.
		- 만약 상속받는다면 부모의 값은 계수처럼 동작해서 이 값에 곱해진다.

![[Pasted image 20250729172819.png]]
> `Crack`의 가운데에 있는 어두운 부분이 `CrackAB`로 구현된 부분이다. `Lifetime`이 더 길기 때문에 `Crack`이 사라지더라도 `CrackAB`는 조금 더 남아있다.

- `Crack AB` 설정 계속
	- `Renderer - Order in Layer`을 `-1`로 설정
		- 위 이미지처럼 보이지 않기 위해(즉 숨기기 위해)
	- `Color over Lifetime`
		- 알파값이 255인 구간을 조금 더 늘렸다.
	- 기타 트릭
		- `Size` : `Additive` 대비 조금 더 크게 설정 (`1.025 ~ 1.05`)
			- 이러면 `Additive`가 나오더라도 살짝 보이겠다

- **알파 블렌드는 기존 이펙트들에 대조를 주기 위해 사용할 수 있다.**
	- `BeamGround`에도 비슷한 작업을 진행함. 
	- `Beam01`에 대한 `AB` 머티리얼 만들고, 크기 키움
	- Tip) `Shader Graph`에서 프로퍼티 할당하는 부분 위에 `Shader Graphs`라고 되어 있는 부분은 이름을 바꿀 수 있다. 내가 만든 쉐이더 그래프는 다 거기에 넣는 식으로 하면 나중에 찾기 편하겠죠?

- 여기서 다룬 알파 블렌드는 정리해보면
> [!tips]
> 기존의 `Additive` 이펙트를 기반으로 구현하되,
> - `lifetime`이 더 길다.
> - 검정색, 알파 100%
> - `renderer - order in layer` : `-1`
> - 사이즈도 조금 더 크게 
>
>정리하면 **밝은 이펙트를 더 강조하기 위해 대조를 사용할 때 `Alpha Blended` 쉐이더를 쓴다**는 것이다. 기존의 밝은 이펙트를 가리지 않고 배경에 나타나도록 구현한다. 이펙트 자체에 어두운 색이 있기 때문에 밝은 배경에서 사용되더라도 이펙트를 보여줄 수 있다.

## 불 이펙트 버전
- 현재 구조
![[Pasted image 20250729175233.png]]

- 이제 이를 기반으로 이펙트들의 여러 가지 버전을 만들어본다.

![[Pasted image 20250729175535.png]]
> 색깔 놀이 : `Anticipation`

![[Pasted image 20250729175904.png]]
> 색깔 놀이 : `Climax`

![[Pasted image 20250729175946.png]]
> 주로 사용된 색상은 이런 계통임


여기에 추가로 이제 불과 관련된 텍스쳐를 만들어 본다.
![[Pasted image 20250729180037.png]]

> 이전엔 레이어 복사해서 가우시안 블러를 적용하지 않았는데 이번에는 사용해봄 ![[Tut_Fire (2).png]]

- `Anticipation`에 있는 `ParticiesIn`을 복붙해서 `FireIn`이라는 걸 만듦
	- 대충 정리하면
	- 파티클 크기 크게, 속도는 조금 더 작게, 파티클 수 적게
	- `Rotation Over Lifetime`도 설정
	- `Shape`의 반경도 작게, `Thickness`도 `0.1`로 설정해서 밖에서 안으로 들어오게끔 구성

- 마찬가지로 `ParticlesOut`도 설정
![[Pasted image 20250729182707.png]]

> - 뭔가 ㅋㅋㅋㅋㅋ 과자들 날라가는 것 같은 느낌도 있긴 하다
> - 텍스쳐를 만들 때 **저 모양 처리를 잘 하는 게 중요한 것 같음.** 

## Nature 버전 - 파트 1
![[Pasted image 20250730154837.png]]
이 2가지 색을 주로 써서
![[Pasted image 20250730154905.png]]
이런 느낌의 이펙트를 만들었다. 무슨 색을 어디에 넣을지는 마음대로 하면 됨

- 여기에 추가로 자연을 나타내는 텍스쳐를 추가한다.
- 이미지 검색으로 잎의 형태를 참고로 작업해도 ㅇㅋ
![[Pasted image 20250730155257.png]]

> 이번에 시험삼아서 AI로 테스트해봤는데
> - `ChatGPT`![[LeafTexture (2).png]]
> - `Gemini`![[unnamed (2).png]]
> ...이다. **`ChatGPT`는 꽤 만족스러운 결과물을 보여주고 있다.** 물론 은은한 블러 효과가 바깥쪽에 살짝 있어서 이걸 다듬을 필요는 있다.
> - 하지만 학습 과정에서는 직접 그려보는 게 더 좋지 않을까라는 생각도 든다. 텍스쳐가 어떻게 이펙트에 반영되는지를 보려면 그런 과정도 거쳐야 할 것 같음. 작업 속도가 빠를 필요성이 있다면 AI에게 의존하는 게 좋겠지만.
> - 참고로 프롬프트는`투명한 배경의 잎 모양 텍스쳐를 하나 만들어주세요. 배경은 투명, 이미지는 여러 알파값이 섞인 흰색으로만 구성되어야 합니다.`으로 전달했다.
![[Pasted image 20250730160706.png]]
> 머티리얼 만들어서 적용해본 모습. 꽤 만족스러운 결과물이긴 하다. 
![[Tut_Leaf (2).png]]
> 직접 작업한 이미지
![[Pasted image 20250730162703.png]]
> 결과물

- 다른 파트에서 작업하지 않은 내용만 정리함
- **`Shape`는 `Circle`로 설정된 상태**
- `Velocity Over Lifetime`
	- 축 별로 설정할 수 있음
	- `Space : World`로 변경
	- `Linear - Y` 그래프를 +5에서 시작해서 -5에서 끝나도록 만듦
	- 이러면 **이펙트가 나타나야할 위치에서 시작해서 위로 튀어오른 후에 다시 내려오는 효과**가 나옴.
![[Pasted image 20250730163622.png]]

- `Climax`의 `LeafsOut`
	- `ParticlesVertical`과 유사하게 동작
	- 대신 원 반경이 조금 더 넓다는 느낌?
![[Pasted image 20250730164152.png]]
- `Noise`도 추가한다.
![[Pasted image 20250730164443.png]]
이런 설정일 때, 올라가는 `Leaf` 이펙트에 흔들림이 추가됨.

## Nature 버전 - 파트 2
- 이펙트가 발생할 때, 원기둥 안에 **나선형으로 솟구쳐 올라가는 듯한 이펙트**를 추가해보자.

### 블렌더
1. `Plane`을 만듦
2. `Add Modifier - Screw` 적용

![[Pasted image 20250730164857.png]]

3. `screw` 옵션 15m로 적용
4. `Angle` 옵션은 회전하는 정도를 적용할 수 있음 - 360 유지
5. `Steps Viewport` - 지오메트리 생성량. 16으로 유지.

6.  현재 `Edit Mode`에서 이 도형의 버텍스를 잡아보면 여전히 `Plane`임.
7. `Object Mode`로 변경 -> 우클릭 -> `Convert To Mesh` 클릭. 
	- `Edit Mode`에서 봤을 때 생성된 `Screw` 전체가 버텍스로 잡혀야 함
8. `Face` 선택 모드에서 한 면을 `Alt + 클릭`하면 해당 도형과 이어진 면들이 선택됨
![[Pasted image 20250730165307.png]]
9. 이 상태에서 `Ctrl + I`를 클릭하면 나머지 부분이 선택된다. `Delete 버튼 - Face`로 제거해준다.
![[Pasted image 20250730165406.png]]
10. 이름을 `Spiral01`로 변경.
11. `U 클릭 -> Unwrap`
	- 옵션은 크게 상관 없어보임
12. `UV Editing`에서 생성된 `UV Map`을 아래처럼 펼침. 단순히 `G`와 `S`의 조합만으로 가능함.
![[Pasted image 20250730165734.png]]
13. `Add Modifier - Subdivision Surface` 추가
	- 12까지 작업했을 때 폴리곤의 수가 적어서 `Edge`가 많아보인다. 이건 보기 좋지 않음.
14. 내보내기 - `Sprial01.fbx`로 저장
	- `Selected Object, Mesh` 옵션 체크.

### 텍스쳐 만들기
- 우선 블렌더에서 `UV - Export UV Layout`으로 `UV Map`을 내보낼 수 있다.
- 강의에서는 이렇게 내보낸 이미지를 포토샵에 넣고, 그 위에서 UV 작업을 진행했음.
![[Pasted image 20250730170528.png]]
> 요런 느낌의 이미지를 만들면 된다~

- 작업 과정을 보면 작은 `Opacity`를 가진 브러쉬들로 큰 것에서부터 시작해서 작은 것으로 그려나가면서 하나씩 겹쳐나간다는 느낌.
- 프로크리에이트에서 이걸 구현한다면 각각을 다른 레이어에서 작업해야겠다.
- 강의에선 위 / 아래를 연결하지 않았다. `Seamless`가 아니기 때문.
- 그래서 그림을 그릴 때는 알파 = 100%인 브러쉬가 없으며, 형태를 다 잡은 후에는 포토샵에서 `Outer Glow`을 적용한다. 
 ![[Spiral01 (2).png]]
- 이런 느낌.

### 유니티
- `SpiralOut`이라는 파티클 시스템을 만든다.
	- `Climax`의 `Cylinder`을 복붙해서 `Spiral`을 만들고 메쉬 할당, 머티리얼도 `Impact01_Add`을 복붙해서 텍스쳐 할당.
- `Model` 자체는 `Scale`을 10으로 해서 옮겼는데, 이 경우 너무 커 보임
	- `3D Start Size`을 `1, 10, 2.5`로 변경
- 나선의 한 방향이므로 **`Start Rotation`만 180도 돌린 다른 나선도 하나 생성해줌**
	- 여기서 방향을 돌린다는 건 `Particle System`의 `Start Rotation` 값임.
	- 나머지는 직접 만져보면서 작업하면 된다.
	- 회전을 늘려도 되고`Rotation Over Lifetime` 자체의 `Lifetime`을 늘려도 되고...
![[Pasted image 20250730190313.png]]

## 셰이더 그래프 : Scroll & Distortion Shader
- 원래 다음 챕터에서 다룰 내용인데, 마지막 수정 전에 해당 셰이더를 여기서도 적용한다고 해서 먼저 작성한다.
- `ParticlesAddtivie_HDR` 셰이더를 복붙, **`ParticlesAddScroll_HDR`로 수정**

### 스크롤 구현
- 원래는 `Tiling And Offset`을 써서 `Time * MainSpeed(Vector2)` 값을 `Tiling And Offset`의 `UV`에 연결하는 방식이었음.
- 여기선 노이즈까지 구현할 예정이라서 `UV`를 별도로 구현한다.
	- 이 경우 위의 `Time * MainSpeed`를 `UV`에 더하는 방식으로 스크롤을 구현할 수 있음.
### Distortion 구현
- 프로퍼티를 추가한다 : `DistortionAmount, float`
	- `Slider`, `0~1`
- `Lerp` 노드에 대한 설명
	- 인풋 A, B, T를 받으며 0~1인 T에 따라 A와 B 사이의 보간된 값을 반환함
	- T = 0이면 A, T = 1이면 B.
- 스크롤에서 구현한 UV를 A에, DistortionAmount를 T에 넣는다. 
- B에는 노이즈 텍스쳐를 넣을 거다
- 셰이더 그래프에서 지원하는 기본 노이즈 텍스쳐의 종류
	- `Simple Noise`
	- `Gradient Noise`
	- `Voronoi`
- 여기선 `Gradient Noise`를 사용함.

![[Pasted image 20250730192046.png]]
여기서 윗 부분은 텍스쳐의 `MainSpeed`를 적용한 부분이고, 아랫부분은 `NoiseSpeed`를 적용한 부분이다.
![[Pasted image 20250730192134.png]]
> 기본 `Flare` 텍스쳐 기준, `DistortionAmount = 0.1`일 때 이런 모양으로 스크롤이 우->좌로 `MainSpeed`에 따라 흘러간다.
> 스크롤의 속도는 `MainSpeed`를, `Noise`가 자글거리는 정도는 `NoiseSpeed`를 따름.


- 추가로 `Gradient Noise`의 `Scale`에 적용할 `Noise Scale`도 설정할 수 있다.

여기까지 만들어놓고 다시 기존 강의로 돌아감. 

## 최종 수정
- 처음에 작업했던 `vfx_AreaOfEffect_v1`을 `v2`로 복사해서 하나 생성해준다.
- `Cylinder`에 위에서 작업했던 셰이더 그래프를 적용해줌. `MainSpeed`와 `NoiseSpeed` 값이 들어가면 된다.

>[!warning]
>- 내 경우 `MainSpeed, NoiseSpeed` 값이 적용되지 않는 듯한 이슈가 있음.
>- 강의에선 파티클 시스템을 멈췄을 때에도 기둥이 흔들리는 효과가 있는데 내 꺼에선 그냥 멈춰있다. 노이즈 자체는 잘 적용되고 있음. 
>- 저런 UV가 이동하는 효과 자체가 적용되고 있지 않은 듯? 이유는 모르겠다. 일단 진행.
>- 방법을 찾아보려고 했는데 `씬 뷰`에서 `Always Refresh`를 활성화하면 텍스쳐가 흐르는 효과가 나타나는데 `게임 뷰`에는 적용되지 않는다. `HDRP`에서만 발생하는 현상이라는 말도 있으니 그냥 그런가보다 하고 넘겨야겠다.

- 그 외 수정사항 : **`Cylinder`와 `Particle`에 `Alpha Blended`이 적용된 이펙트들이 추가됨.** 
	- 이 구현은 렌더링 순서를 -1로 지정하는 것도 물론 가능하지만, 일반적인 파티클들의 경우 단순히 조금 더 적은 수의 검은 파티클들을 추가하는 것으로도 가능하다. 
	- `Anticipation`
		- `ParticiesIn`에 파란 계열의 파티클을 추가한다.
		- 저 파란 색은 `Cylinder`의 `Color over lifetime`에서도 시작색으로 적용한다. 가장 뚜렷한 시점의 색은 기존의 분홍-보라 사이의 그 색깔.
		- 이를 위해 `Cylinder`의 `Start Color`는 흰색으로 변경한다. **색상을 적용할 때는 둘 중 하나만 써야 하는 듯.**
		- 대조를 강화하기 위해 `BeamGroundAB`의 파티클을 하나 더 추가한다.
		- `CylinderAB`도 만든다. `Impact01_ADD`를 복붙해서 `Impact01_AB`을 추가함. 
			- `Order In Layer : -1`
			- `Start Color : Black, Alpha: 100%`
			- 그대로 구현하면 잘 안보이니 기존 이펙트 대비 z축을 늘린다든가 하는 식으로 배경을 감싸도록 할 수 있음.
		- `ParticlesIn`에 대해서도 대조를 줄 수 있다.
			- `Beam01_AB` 적용
			- `ParticlesIn`보다는 마지막 파티클 부분이 조금 더 적게
			- `Hemisphere`의 반경도 조금 더 크게 줬음.
	- `Climax`

- 최종
- `Anticipation`
![[Pasted image 20250730200417.png]]

- `Climax`
- ![[Pasted image 20250730200449.png]]
- ![[Pasted image 20250730200511.png]]