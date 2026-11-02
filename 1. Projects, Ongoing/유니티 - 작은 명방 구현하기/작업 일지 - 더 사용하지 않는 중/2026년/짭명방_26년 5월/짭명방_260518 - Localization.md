
>[!note]
>- 목표 : 언어 전환 기능 만들기
>- `LocalizationManager`가 있는 김에 다국어가 어떻게 관리되는지 테스트해보고 적용해보면 좋을 것 같아서 해봄

## 현재까지의 다국어 구현
- 일단 언어 전환 기능을 직접적으로 넣은 건 아님. 

>[!note]
>- `enum` 타입으로 `Language`를 구현, `GameManagement`에 위치, `Language`의 인덱스에 따라 현재 나타나는 언어가 결정되는 방식.
>- 텍스트가 나타날 곳에 `LocalizationManager`에서는 `GetText(string key)`를 통해 테이블에서 해당하는 키의 값을 찾아서 집어넣는 방식.
>	- 아직까지는 전체적으로 언어 전환 기능을 넣은 게 아니라서, 한글 텍스트가 직접 들어가 있는 UI도 많다.
>- 테이블은 `LocalizationTable`이라는 이름의 `SO`로 관리되어 왔다. 각 항목은 `Key`와 `List<string>`인 `Translation`으로 관리되어 있음.

- 기존 `LocalizationTable`의 필드 예시
![[Pasted image 20260518145805.png]]

### SO 방식의 한계점
- AI가 던져준 내용들. (SO로 쓰는 경우도 여전히 있다고 함)
1. 필드 순서가 `enum` 타입에 의해 정의되어 있음
2. 번역가가 `SO`에 접근하려면 유니티를 깔아서 프로젝트를 여는 과정을 거쳐야 함
3. 런타임에 동적 추가 불가능 

- 개인적으로 느낀 건 SO를 만들 때 저 `key`값이 어디에 쓰이는지를 잘 모르겠다는 경우가 많았음. 
	- 저걸 만들고 바로 이어붙이면 상관없는데, 나중에 프로젝트를 다시 볼 때 '이거는 어디에 쓰이지?'를 알려면 코드 단위로 뜯어서 봐야 했다는 것.

## Unity Localization Package
- `CSV, JSON` 등등의 방식이 있음
- 유니티에서 공식적으로 지원하는 패키지가 있다고 하니 이걸 써보겠음
- 우선 테스트용 씬을 만들어서 적용해보고 이걸 실제 프로젝트에도 옮기는 식으로 작업

### 세팅
![[Pasted image 20260518153020.png]]
> - 패키지 매니저에서 `Localization`을 검색해서 설치
> - `Project Settings`에서 `Add Locale` 클릭, 사용할 언어들 체크
> 	- `en-US`와 `en`의 차이는 지역 여부로, 지역에 따른 차이를 두지 않을 거고 텍스트 번역만이 목적이라면 2글자 코드로 사용하면 됨(모두 BCP-47 언어 태그)
> - 이후 `Project Locale Identifier`에서 기본 설정 언어 지정

- `Localization`을 위한 파일 세팅
```
Assets
- Localiation/
  - Locales/
  - Tables/
```

### 테스트 - 테이블 생성
- `String Table` 생성 : `Window > Asset Management > Localization Tables`
- `+ New Table Collection`으로 테이블 생성, 경로는 위의 `Tables/`

![[Pasted image 20260518155805.png]]
> 이런 식으로 테스트 필드 2개를 만들어봄


### 테스트 - 필드에 할당
- `TMP` 컴포넌트에 **`Localize String Event` 컴포넌트**를 추가
![[Pasted image 20260518160010.png]]
1. `String Reference`에 적용할 키값 설정
2. `Update String`에 이벤트 추가
![[Pasted image 20260518160155.png]]
> `string`을 업데이트할 항목을 추가하면 됨 : 현재의 `TMP`와 `TMP.text`으로 설정

- 패키지를 추가하면 게임뷰에 우측 상단처럼 언어 설정이 나타난다. 이걸 바꿔가면서 잘 적용되는지 체크하면 됨.
![[Pasted image 20260518160257.png]]
![[Pasted image 20260518160308.png]]

### 추가 이슈 - 일본어 폰트

#### 폰트 가져오기 & Atlas 생성

- 일본어에 해당하는 폰트가 없다. 구글 폰트에서 `NotoSans JP`를 받아옴.
```
- 폴더 구조
static/
ttf 파일
```
> 루트의 `ttf` 파일은 가변 폰트라서 불안정하다고 한다. 
> `static`에서 지정된 굵기의 폰트만 사용하는 게 좋다. 일반적으로 `Regular`을 사용.

- 폰트를 프로젝트의 에셋에 넣고, `Create > TMP > Font Asset > SDF`로 폰트 에셋 생성.

- 상단 탭 `Window > TMP > Font Asset Creator`
	- `Character set` : `Unicode Range`
	- `Character Sequence : 3000 - 9FFF`(일본어 범위 포함)
	- `Atlas Resolution` : `4096 x 4096`
		- 글자 수가 많아서 아틀라스를 크게 잡아야 함

- 이렇게 만든 아틀라스 파일을 저장한다. **이 파일은 별도로 참조하지 않아도 폰트의 `SDF` 파일이 참조하고 있음.** 

#### Asset Table 생성
- 언어 단위로 폰트를 교체할 수 있음
- `Window > Asset Management > Localization Tables > New Table Collection > Asset Table로 타입 변경`
	- 위에서 `Localization Table` 만드는 과정이랑 동일하다. 마지막에 타입만 바꿔주면 됨.
- 테이블의 각 언어에 해당하는 폰트의 `SDF` 파일을 연결해주면 된다.


### TMP의 디폴트 세팅 변경
- TMP에는 자체적인 폰트 세팅이 되어 있기 때문에, 이걸 언어에 따라 설정을 바꿔줘야 할 듯
- 이 부분에서 막혔다. 언어를 바꿀 때마다 모든 TMP의 폰트를 바꿔줘야 하나?

