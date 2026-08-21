1. [[#프로젝트 정리|프로젝트 정리]]
2. [[#바닥 회전|바닥 회전]]
	1. [[#바닥 회전#Rotator 스크립트 준비|Rotator 스크립트 준비]]
	2. [[#바닥 회전#속도와 시간 간격|속도와 시간 간격]]
	3. [[#바닥 회전#Rotate 스크립트 수정하기|Rotate 스크립트 수정하기]]
3. [[#게임 UI 제작|게임 UI 제작]]
	1. [[#게임 UI 제작#생존 시간 텍스트 제작|생존 시간 텍스트 제작]]
		1. [[#생존 시간 텍스트 제작#캔버스, 이벤트 시스템|캔버스, 이벤트 시스템]]
	2. [[#게임 UI 제작#텍스트 배치|텍스트 배치]]
		1. [[#텍스트 배치#앵커 프리셋|앵커 프리셋]]
	3. [[#게임 UI 제작#텍스트 꾸미기|텍스트 꾸미기]]
	4. [[#게임 UI 제작#게임 오버 텍스트, 최고 기록 텍스트|게임 오버 텍스트, 최고 기록 텍스트]]
4. [[#게임 매니저 제작|게임 매니저 제작]]
	1. [[#게임 매니저 제작#GameManager 스크립트|GameManager 스크립트]]
	2. [[#게임 매니저 제작#생존 시간 표시하기|생존 시간 표시하기]]
	3. [[#게임 매니저 제작#게임 재시작 구현|게임 재시작 구현]]
	4. [[#게임 매니저 제작#EndGame() 구현|EndGame() 구현]]
	5. [[#게임 매니저 제작#PlayerPrefs|PlayerPrefs]]
	6. [[#게임 매니저 제작#최고 기록 저장 / 읽기 구현|최고 기록 저장 / 읽기 구현]]
	7. [[#게임 매니저 제작#PlayerController에서 EndGame() 실행하기|PlayerController에서 EndGame() 실행하기]]
	8. [[#게임 매니저 제작#게임 매니저 오브젝트 설정|게임 매니저 오브젝트 설정]]
5. [[#빌드하기|빌드하기]]

## 프로젝트 정리
- `Project` 탭에서..
- `Scripts, Materials, Prefabs` 폴더를 만들어 파일들을 넣어둔다

## 바닥 회전
- 게임의 난이도 높이기 : 바닥을 일정 속도로 회전시킨다.

### Rotator 스크립트 준비
- `Scripts > Create > C# > Rotator.cs` 생성
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Rotator : MonoBehaviour
{

    public float rotationSpeed = 60f;

    void Update()
    {
        transform.Rotate(0f, rotationSpeed, 0f);
    }
}
```

> `Rotate(float xAngle, float yAngle, float zAngle)`
> Update() 메서드에 의해 매 프레임마다 실행되므로, 매 프레임마다 60도 회전한다.
> - 현 상태는 한 번에 60도를 회전하는, 문제가 있는 코드이다. 
> - 즉 `Rotate()`는 1초에 60도를 돌리는 게 아니라 실행될 때마다 60도를 돌리는 코드임

- 스크립트 적용 : `Level` 오브젝트에 드래그 & 드롭

### 속도와 시간 간격
- 화면 갱신 주기는 컴퓨터 성능에 의존적이므로, `Update()`가 실제로 1초에 몇 번 실행될지는 알 수 없다.

- **프레임 제한 해제**
	- 콘솔은 사양이 정확히 정해져 있기 때문에, PC 대비 FPS를 정확히 추측할 수 있다.
	- 몇몇 콘솔 게임은 PC로 출시할 때, 60FPS를 그대로 사용하기도 한다.
	- 프레임 제한을 강제로 해제하면, 고정 프레임을 사용하는 게임 중 일부는 속도가 몇 배 빨라지거나, 물리 상호작용이 비현실적으로 변경되는 경우가 있다. 

> 예를 들어 고정 프레임이 60인 경우, 1초 동안 1m를 이동시키겠다면 Update() 한번에 1/60을 지정하면 됨. 근데 프레임 제한이 해제된다면? 1초에 2m를 가게 된다. 

- 따라서 FPS가 다른 경우를 고려해야 한다. 

- ★ 이를 해결하는 방법은, **시간 간격으로 쪼개서 누적하는 것**이다.
1. FPS의 역수를 회전 값에 곱한다. 
	- 이 경우 1/60은 직전 프레임과 현재 프레임의 시간 간격이 된다.
2. 해당 값을 원래 넣으려고 했던 값 60도에 곱한다

> 계산 1) 60도 회전 * 1/60 * 1초에 60번 실행 = 총 60도 회전
> 계산 2) 60도 회전 * 1/120 * 1초에 120번 실행 = 총 60도 회전

이는 원래 값을 쪼갠 다음, 누적해서 더한다고 이해할 수도 있다. 

위 방법은 Update 메서드의 실행 횟수에 관계 없이 적용할 수 있다.
이전 장에서 `Time.deltaTime` : 이전 프레임과 현재 프레임 사이의 시간 간격 을 얘기했는데, 이 값이 프레임의 주기이자 `1/FPS` 값이다. 

### Rotate 스크립트 수정하기
```cs
    void Update()
    {
        transform.Rotate(0f, rotationSpeed * Time.deltaTime, 0f);
    }
```

## 게임 UI 제작
- 생존 시간, 게임 오버, 최고 기록 등을 표현하는 UI를 만든다.
- 유니티 이전의 게임 엔진은 월드와 UI를 별도의 공간으로 다뤘다.
- 반면, 유니티의 UI 시스템`UGUI`는 UI 요소를 게임 월드 속의 게임 오브젝트로 취급한다.
	- 하나의 UI 요소는 씬 속의 하나의 게임 오브젝트가 된다. 
	- 이게 생각보다 강력한 장점임

### 생존 시간 텍스트 제작
- `씬 - 2D 버튼 - 2D 편집 모드로 전환`
- `Hierarchy - Create - UI - Text 클릭`
	- 24년 6월 기준, Text는 `Legacy`에 있음
- `Hierarchy - Canvas 더블 클릭 - 씬 창에서 Canvas 포커스`

- `Text` 게임 오브젝트를 만들면 총 3개의 게임 오브젝트가 생성된다 : `텍스트Text`, `캔버스Canvas`, `이벤트 시스템EventSystem`

#### 캔버스, 이벤트 시스템
- `Text` 게임 오브젝트는 `Canvas` 게임 오브젝트의 자식이다. UI요소는 캔버스의 2차원 평면에 배치되기 때문이다.
- `UI 게임 오브젝트`는 일반 게임 오브젝트와 달리 트랜스폼 컴포넌트를 확장한 `사각 트랜스폼Rect Transform` 컴포넌트를 가진다. 
- 즉, UI요소는 자신이 배치될 액자가 필요하고, Canvas 게임 오브젝트는 액자 역할을 한다.
- 따라서 **UI 게임 오브젝트를 만들 때, Canvas 게임 오브젝트가 없다면 자동으로 하나 생성**된다.
- 모든 **UI 게임 오브젝트는 Canvas의 자식**이다.

- `EventSystem` 게임 오브젝트는 이벤트 시스템 컴포넌트를 가진 게임 오브젝트로, UI 게임 오브젝트에 클릭, 터치, 드래그 같은 상호작용을 이벤트 메시지로 전달한다.
	- 이게 없으면 UI 버튼이나 슬라이드바 클릭, 드래그 등의 UI 상호작용을 할 수 없다.
	- 직접 만질 일은 거의 없음.

### 텍스트 배치
- UI 게임 오브젝트를 정렬할 때는 앵커프리셋을 사용한다.
- `Hierarchy - Text - Time Text로 이름 변경`
- `Inspector - Anchor Preset - [Alt] + top-center 클릭`
	 - 앵커 프리셋은 `Rect Transform`의 좌측 상단에 있는 `top, center` 표시된 그림임

#### 앵커 프리셋
- UI요소를 잘 배치하기 위해서는 앵커, 피벗, 포지션 값을 조정해야 한다.
- 앵커 프리셋을 연 상태에서 **`Alt`키를 누르면 `스냅핑Snapping`이 활성화**된다. 앵커값 외에도 **포지션 값도 변경**되며, UI 게임 오브젝트가 해당 방향의 모서리에 달라붙는 형태로 정렬된다.

### 텍스트 꾸미기
- `Time Text`의 텍스트 변경하기
	- `Time Text` 게임 오브젝트의 `Text` 컴포넌트의 `Text` 필드 내용을 `Time : 0`으로 변경
	- `Text` 컴포넌트의 `Alignment`를 `Center, Middle`로 변경
	- `Color 필드 클릭 - 폰트 컬러 255, 255, 255로 변경`

- 폰트 크기 키우기
	- `Rect Transform`의 `Pos Y`를 `-30`으로 변경
	- `Text` 컴포넌트의 `Font Size` `42`로 변경
	- `Horizontal Overflow, Vertical Overflow`를 `Overflow`로 변경
> - 폰트 크기를 42로 키우면 글자가 글 상자를 넘쳐 잘려서 텍스트가 보이지 않게 된다. 이러한 상태를 글이 글 상자를 넘치는 상황이라 오버플로우라고 함
> - 수평, 수직 오버플로우는 글상자를 넘칠 경우 해당 방향으로의 글자를 잘라낼지 표시할 것인지를 결정한다.
> - 수평의 기본 설정 `랩핑Wrap` 은 강제로 줄바꿈을 적용한다.
> - 수직의 기본 설정 `자르기Truncate`는 글자가 수직 방향으로 넘칠 때 잘라낸다.

- 그림자 효과 추가
	- `Shadow` 컴포넌트 추가 `Add Component > UI > EFfects > Shadow`


### 게임 오버 텍스트, 최고 기록 텍스트
- `Time Text` 클릭 후 `Ctrl + D`로 복제, 이름을 `Gameover Text`로 변경
- `Text 필드 값`을 `Press R to Restart`로 변경
- `앵커 프리셋`을 열고, `Alt`를 누른 상태에서 `middle-center` 클릭

- Record Text 만들기
	- 마찬가지로 진행,
	- Pos Y = -40
	- Text 필드 : Best Record : 0
	- Font Size 30

- Record Text를 Gameover Text의 자식으로 만들기 : 드래그 & 드랍
- 이후 Gameover Text의 체크박스를 해제해서 비활성화한다. 게임오버했을 때만 떠야 하니까


## 게임 매니저 제작
- 게임 규칙, 게임 오버 상태 표현, 생존시간 수치 관리, 게임 UI 갱신 등

### GameManager 스크립트
- 아래의 스크립트를 작성한다.
- `GameManager.cs`
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI; // UI 라이브러리
using UnityEngine.SceneManagement; // 씬 관리 라이브러리

public class GameManager : MonoBehaviour
{
    public GameObject gameoverText; 
    public Text timeText;
    public Text recordText;

    private float surviveTime;
    private bool isGameover;

    // Start is called before the first frame update
    void Start()
    {
        surviveTime = 0;
        isGameover = false;

    }

    // Update is called once per frame
    void Update()
    {
        
    }

    public void EndGame()
    {

    }
}
```

 >public GameObject gameoverText; 
> public Text timeText;
> public Text recordText;
>- 각각의 타입이 다름에 주목하자 
	- 텍스트 컴포넌트의 **텍스트 내용을 바꾸고 싶다면, `Text` 타입의 변수에 할당**한다. 그리고 `Text` 타입의 `text` 필드에 접근해서 수정한다. 이 필드는 인스펙터 창에서 편집했던 `Text` 필드와 동일함. 
	- 한편, **게임오버를 표시하는 `Gameover Text`는 표시할 텍스트 내용이 변경되지 않는다.** 활성화/비활성화 하는 방식으로만 사용하므로 `Gameobject` 타입으로 선언한다.

다른 내용은 특이한 거 없으니까 책에서 다뤘더라도 넘어감

### 생존 시간 표시하기
- GameManager 스크립트에 생존 시간 측정 및 표시하는 기능을 추가한다.
- `Update()` 메서드에 생존 시간을 `Time.deltaTime` 값을 누적해서 더하며 표현한다.
- 그 다음, `timeText`에 할당한 텍스트 컴포넌트의 `Text` 필드에 `surviveTime` 값을 이용해 생존 시간을 표시한다.
```cs
    void Update()
    {
        if (!isGameover)
        {
            // 생존 시간 갱신
            surviveTime += Time.deltaTime;
            // 갱신한 생존 시간 텍스트 컴포넌트에 표시
            timeText.text = "Time : " + (int)surviveTime;
        }   
    }
```
> 인스펙터 창에서 표시되는 컴포넌트의 공개된 필드 대부분은 코드 상에서 접근하고 수정할 수 있다. 이런 것들은 대부분 `public`으로 선언된 변수들임. 그리고 `.`으로 변수에 접근할 수 있다.
> - 최초에 작성했던 게 `Time : 0`이었다. 위에 작성한 코드는 이걸 지우고 `Time : (int)surviveTime`을 할당한 것.
> - 여기서 **`(int)surviveTime`은 `float` 타입의 `surviveTime`을 `int`로 형변환한 것**이다. 소수점 아래를 그대로 표현하는 사태를 막기 위해 저렇게 표시한 것이다.

### 게임 재시작 구현
- `isGameOver = true`일 때, R 키를 눌러 현재 씬을 다시 로드한다
	- 현재 씬을 다시 로드한다 = 다른 게임 월드로 전환한다
	- 기존 씬의 게임 오브젝트 전부를 파괴한다 => 이를 응용하면 현재 활성화된 씬을 다시 로드하는 방식으로 게임 재시작을 구현할 수 있다.
	- 따라서, 지금 프로젝트에서 게임 재시작을 구현하려면 현재 활성화된 `SampleScene` 씬을 다시 로드하면 된다.
```cs
    void Update()
    {
        if (!isGameover)
        {
            // 생존 시간 갱신
            surviveTime += Time.deltaTime;
            // 갱신한 생존 시간 텍스트 컴포넌트에 표시
            timeText.text = "Time : " + (int)surviveTime;
        }   
        else
        {
            if (Input.GetKeyDown(KeyCode.R))
            {
                // SampleScene 씬 로드
                SceneManager.LoadScene("SampleScene");
            }
        }
    }
```
> - `SceneManager` : `using UnityEngine.SceneManagement`에서 가져온 유니티 내장 씬 관리자이다. `LoadScene()`은 입력으로 씬의 이름을 받아 해당 씬을 로드한다.
> - **`LoadScene()`으로 게임을 재시작하는 것과 같은 효과**를 낼 수 있다.
> - 여기서 로드할 씬은 빌드 설정의 빌드 목록에 있어야 한다. `SampleScene`은 빌드 목록에 자동 등록되어 있기 때문에 따로 빌드 목록에 추가할 필요는 없다. 빌드 설정 창과 빌드 목록은 `File > Build Settings...`로 확인할 수 있다.

### EndGame() 구현
- 현재 게임을 게임 오버 상태로 만드는 메서드. 
- 플레이어가 죽을 때 실행되며, 현재 게임 상태를 게임오버 상태로 변경하고 게임오버 시 필요한 처리를 실행한다.
- 아래의 기능을 가진다.
	- `isGameOver = true`로 변경
	- 현재 생존 시간 기록과 최고 생존 시간 기록 비교
	- 게임오버 UI를 활성화하고, 최고 기록 표시.

- 이 메서드는 플레이어가 호출해야 한다. 플레이어가 죽었을 때, `PlayerController` 스크립트에서 `GameManager` 컴포넌트로 접근, `EndGame()` 메서드를 실행한다.
```cs
    public void EndGame()
    {
        isGameover = true;
        gameoverText.SetActive(true);
    }
```
> 이 다음, 아래의 과정을 거친다.
> - 이전 저장 기록을 불러와 현재 생존 시간과 비교
> - 최고 기록을 `recordText`에 할당된 텍스트 컴포넌트로 표시
> - 갱신된 최고 기록을 저장

최고 기록의 수치를 저장하여 프로그램 종료 후에도 유지하고 다시 불러와 사용하는 처리는 `PlayerPrefs`로 구현할 수 있다.


### PlayerPrefs
- `플레이어 설정Preference`이라고 읽으며, 어떤 수치를 로컬에 저장하고 다시 불러오는 메서드를 제공한다.
- `키-값` 단위로 데이터를 로컬에 저장한다. 값을 저장할 때 사용할 키를 기억해 나중에 다시 저장된 값을 가져온다.
- `SetInt, SetFloat, SetString` 등으로 키(string) - 값(int, float, string)을 저장할 수 있음.
- 또, `GetInt, GetFloat, GetString` 등으로 가져올 수도 있다.
	- 여기서 아무 값도 없다면 숫자 자료형은 0을, 문자 자료형은 ""을 가져온다.
- 이외에도 `HasKey()`로 해당 키로 저장된 값이 있는지 여부를 볼 수도 있다.

### 최고 기록 저장 / 읽기 구현
- `EndGame()` 메서드를 아래처럼 완성한다.
```cs
    public void EndGame()
    {
        isGameover = true;
        gameoverText.SetActive(true);

        float bestTime = PlayerPrefs.GetFloat("BestTime");

        if (surviveTime > bestTime)
        {
            bestTime = surviveTime;
            PlayerPrefs.SetFloat("BestTime", bestTime);
        }

        recordText.text = "Best Time: " + (int)bestTime;
    }
```

### PlayerController에서 EndGame() 실행하기
- `PlayerController.cs`의 `Die()`를 아래처럼 수정한다.
```cs
    public void Die()
    {
        // 자신의 게임 오브젝트 비활성화
        gameObject.SetActive(false);

        // 씬의 GameManager를 찾아 가져온 다음 EndGame() 메서드를 실행해 게임을 끝냄
        GameManager gameManager = FindObjectOfType<GameManager>();
        gameManager.EndGame();
    }
```

### 게임 매니저 오브젝트 설정
- `Hierarchy > Create > Create Empty, 이름 Game Manager`
- `GameManager.cs`를 드래그 & 드랍

- GameManager 컴포넌트 설정
	- `Hierarchy`의 `Gameover Text` 를 `Game Manager` 오브젝트의 `Gameover Text` 필드에 드래그 & 드랍
	- `Time Text, Record Text` 모두 같은 작업을 해주자.

- 플레이 버튼을 눌러 테스트해본다
	- 죽었을 때 생존 시간이 더 이상 갱신되지 않는지
	- R키를 누르라는 게임오버 안내 텍스트가 나오는지
	- R키를 눌렀을 때 재시작되는지 등등을 체크한다

## 빌드하기
- 유니티 프로젝트 폴더 **외부**에 새 폴더 만들기
	- 빌드 경로에 제한은 없으나, 입문자가 프로젝트 폴더 내에 빌드 파일을 저장하면 프로젝트 에셋 파일과 빌드 파일이 뒤섞여 프로젝트가 망가지는 경우가 많다.

- `File > Build Settings > Build and Run (파일 탐색기 실행) > 폴더 선택 후 Save`
- 빌드가 실행되면 완성한 게임으로 진입하지 않고 `유니티 런처`라는 게임 설정 창이 표시된다. 유니티 런처에서 `Play` 버튼을 누르면 게임이 시작된다.

