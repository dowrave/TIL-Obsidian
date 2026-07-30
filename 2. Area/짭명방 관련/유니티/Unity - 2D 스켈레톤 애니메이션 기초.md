
- [동영상 강의](https://www.youtube.com/watch?v=GhwAFTNfqgs&list=LL)
## 준비
1. 포토샵 등에서 레이어 별로 파츠를 분리해 전체적인 그림을 완성한 `PSB` 파일을 만듦
> 내 경우 레이어별로 파츠를 구분해서 작성한 뒤 `Procreate`로 `PSD` 파일로 저장했는데, 이렇게 해도 됨

2. 유니티에서 `Package Manager - Features - 2D` 전체를 다운받음
	- 아마 실제로 필요한 건 아래 정도일 거 같긴 함
		- `2D animation`
		- `2D PSD Importer`

3. 에셋으로 `PSB`(`PSD`) 파일을 옮긴 뒤, `Open Sprite Editor`로 들어간다.
	- `PSB`파일의 경우 별다른 설정 없이 `2D PSD Importer`가 적용되지만
	- `PSD` 파일의 경우 디폴트로 `UnityEditor.TextureImporter`가 적용되어 있다. 이걸 바꿔주면 됨. 
		- 디폴트를 변경하는 방법은 [여기](https://docs.unity3d.com/Packages/com.unity.2d.psdimporter@9.0/manual/PSD-override.html)에 있으니 참고. 

4. `Sprite Editor`에 들어가면 각 레이어별로 분리된 파트들이 있다.

5. `Sprite Editor > Skinning Editor`로 들어가면 레이어들이 합쳐진 이미지가 나오는데, 여기서 `Bone` 설정을 시작한다.

## SkinningEditor에서 작업
![[Pasted image 20241122214716.png]]
1. Create Bone을 클릭해서 몸통의 빨간 부분을 클릭(검은 점 ~ 가늘어지는 부분까지)
2. 이 상태에서 끝나는 지점에서 다시 시작하려고 하는데 우클릭으로 취소할 수 있음
3. 왼쪽 날개 : 몸과 연동시키기 위해, 빨간 Bone의 검은 부분을 클릭함
4. 왼쪽 날개의 검은 점 부분을 클릭하면 저렇게 반투명한 노란 선이 만들어짐. 이 Bone은 날개 끝까지 이어줌. 이 작업은 Bone의 부모 - 자식 관계를 설정하는 것이다.
5. 오른쪽 날개도 몸통의 Bone을 부모로 해서 3, 4번처럼 작업함.

6. 뼈를 붙이기에 앞서, 우선 `Mesh`를 나눠야 한다. 실제로 움직이는 게 `Mesh`라서 나눠준다. 
	- `Auto Geometry`를 클릭
![[Pasted image 20241122215055.png]]
- 우측 하단에 `Weights`라는 옵션이 있다. 
	- 이를 활성화하면 뼈도 메시에 붙는데, 뼈는 따로 동작하는 게 좋음. 그래서 체크 해제.
	- `Generate For All Visible` 클릭.
![[Pasted image 20241122215231.png]]
![[Pasted image 20241122215304.png]]
- 관련 옵션
	- `Outline Detail` : 스프라이트의 외곽선에 들어가는 점의 갯수
	- `Alpha Tolerance` : 투명도를 취급하는 스프라이트 기준.
	- `SubDivide` : 내부 삼각형(메시)의 갯수
	- 전체적으로 크게 건드릴 일이 없지만, 머리카락은 디테일을 추가헤 작게 만드는 게 좋다.
- 예를 들어 42, 55, 53으로 설정하면 아래 같은 느낌이 됨.
![[Pasted image 20241122215449.png]]

7. 뼈를 붙이는 작업을 진행한다. `Bone Influence`창을 연다.
	- `Sprite`에 어떤 `Bone`이 영향을 받을지를 설정한다.
	- 클릭 1번은 `Bone` 설정, 더블클릭은 `Sprite` 설정.
	- 예를 들면 왼쪽 날개에는 왼쪽 Bone이 될 거임
	- 이런 식으로 각각의 스프라이트에 각 뼈대를 연결해준다.

- 이 상태에서 `Preview Pose`를 누르고 뼈를 움직이면 움직이지 않음. 연결하겠다고 지정한 상태이고, 실제로 연결되지는 않았음. 최종적으로는 무게까지 설정해야 실제로 움직인다.

8. 배경을 더블클릭한 다음 `Auto Weights`를 클릭.
	- 스프라이트가 클릭된 상태라면 해당 스프라이트에 대해서만 Weights가 생긴다.
	- 따라서 배경을 더블클릭한 상태에서 `Generate All`을 눌러준다.

- 다시 `Preview Pose`를 누르고 뼈를 움직여보면 스프라이트가 Bone의 움직임에 따라 움직이는 걸 확인할 수 있다.
![[Pasted image 20241122220137.png]]

9. **`Apply`를 누르면 리깅 작업이 완료된다.**

## 애니메이션 작업
1. 작업 결과물을 씬으로 옮긴다.
2. 해당 애셋을 보면 `Character Rig`의 `Pivot = Bottom`으로 되어 있다.
![[Pasted image 20241122220422.png]]
- 스프라이트의 아랫쪽 중심을 축으로 한다는 의미임. 
- 씬 뷰에서 `Center`을 `Pivot`으로 바꿔서 보면 이해가 갈 거임
![[Pasted image 20241122220629.png]]
- 따라서 저 `Pivot`을 `Center`로 바꿔줌.

3. 컴포넌트와 구성 요소 추가 및 프리팹으로 저장
	1) `Animation Controller` 1개와 `Animation` 2개를 생성한다. 
		- 컨트롤러
		- 애니메이션 1: Idle(가만히 있는 상태)
		- 애니메이션 2: Move
		- 2개의 애니메이션은 반복되므로 `Loop Time` 체크.
	2) `Animation Controller`에 `Idle`, `Move`을 넣는다.
	3) 컨트롤러, 애니메이션 창은 `Window - Animation - Animation / Animator`에 있음.
	4) 캐릭터 오브젝트에 `Animator`을 붙여준다.
		- 앞에서 생성한 `Animatior Controller`을 추가.
		- `Sorting Group`도 추가하는 게 좋다. 캐릭터에 있는 하위 오브젝트들을 하나로 묶어서 스프라이트 정리를 편하게 한다.
	6) 오브젝트를 프리팹으로 저장한다.
![[Pasted image 20241122221253.png]]

4. 애니메이션 구성 시작
	 - 창이 없다면 위에서 말한 `Window - Animation - Animation`으로 들어가고 오브젝트 클릭.
	 - Animation 창에서 Idle로 설정
		 - 좌측 상단의 레코드 버튼을 클릭한 후
		 - 30프레임을 클릭하고
			 - 윗쪽에 프레임 표시되는 부분을 클릭해야 함
			 - `...`에서 시간/프레임 변환 가능
		 - 가장 중심이 되는 Bone을 위로 살짝 올려준다
		 - 그 다음 0프레임의 정보는 60프레임으로 넣어준다. (복붙 가능)
		- 추가로, 날개에 움직임도 줄 수 있다. 여기도 마찬가지로 프레임 설정 후 위치 설정.
	 - 즉 애니메이션을 설정하는 과정은 **프레임 설정 후 위치 설정**으로 이뤄지며, 그 사이는 보간되는 것으로 보임.
	 - 그리고 애니메이션은 `Bone` 단위로 녹화가 되므로, 변경하고 싶은 Bone들을 모두 지정한 뒤 Animation 탭에 잘 올라갔는지 확인하고 작업한다.




> - Scene 뷰에서 혹시 Bone이 보이지 않는다면 `Game > Gizmos` 체크 여부 확인. 체크돼야 함.
 - 또, Scene 뷰에 커서를 올리면 Bone 기즈모가 사라지는 경우, 우측 상단의 `Toggle Visibility of all Gizmos in the Scene View`를 활성화하자.

- 0프레임
![[Pasted image 20241122224619.png]]

- 30프레임
![[Pasted image 20241122224631.png]]

![[Pasted image 20241122224722.png]]
> 애니메이션 탭의 설정

- Move 애니메이션도 움직이는 양을 강화시켜서 구현해보자. 

- 여기선 간단한 캐릭터로 설명했지만, 더 인간에 가까운 캐릭터도 구현이 가능하다. 