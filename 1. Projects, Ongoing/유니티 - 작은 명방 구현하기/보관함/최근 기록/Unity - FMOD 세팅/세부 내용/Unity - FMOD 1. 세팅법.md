#유니티-FMOD

## 기본 세팅(프로젝트 생성부터 유니티 연결까지)
- 2가지가 필요하다.
	- 패키지 : `FMOD for Unity`(유니티 패키지 매니저)
	- 스튜디오 : `FMOD Studio`(웹에서 별도로 설치)
	- 스튜디오와 패키지의 버전은 일치시키는 게 좋다.


### FMOD Studio 세팅
#### 1. **`FMOD Studio`에서 새로운 프로젝트 생성.** 
- 프로젝트는 유니티 프로젝트 외부에 저장

---
#### 2. `뱅크Bank` 빌드 경로 설정 및 빌드
1. `FMOD Studio`에서 `Edit > Prefernces > Build > Built banks output directory`를 유니티 프로젝트의 `/Assets/StreamingAssets/fmod`로 설정
	- 여기서 `StreamingAssets/`, `fmod/`은 수동으로 만들어주자.
2. `F7`로 빌드
	- `Master.bank`, `Master.strings.bank`가 생기는지 확인

> 내 경우 `Desktop`이라는 폴더가 추가로 생겼고 그 안에 `bank` 폴더들이 구성되었음

##### 뱅크 세부 설명
- **뱅크 = FMOD의 사운드 패키지 파일**
- 유니티 기본 오디오와 FMOD의 주요 차이점

| Unity 기본 오디오                   | FMOD                 |
| ------------------------------ | -------------------- |
| `mp3, wav` 파일                  | 이벤트(Event)           |
| `AudioClip`을 `AudioSource`에 붙임 | 이벤트를 뱅크에 담아서 빌드      |
| 파일 그대로 사용                      | `.bank` 파일로 패키징해서 사용 |

> `이벤트` : 개별 사운드 요소
> `뱅크` : 개별 `이벤트`들을 유니티가 읽을 수 있게 압축/포장한 파일
> 	- 특정 뱅크는 특정 씬에서만 로드될 수 있게 구성되기도 한다.

---
#### 3. 세부 설정
![[Pasted image 20260508162539.png]]
1. 왼쪽 `Routing - Add Group`으로 `BGM, SFX` 그룹을 추가해줌
	- 기존 유니티에서 `Audio Mixer Group`으로 `Master, BGM, SFX`가 있었기 때문에 이 설정을 따라감

2. `Window > Event Editor(ctrl + 1)`으로 탭을 전환, `BGM`과 `SFX` 폴더를 만들고 그 안에 `New Event`로 `LobbyTheme`과 `StageTheme`을 추가.
	- 기존 탭은 `Mixer(ctrl + 2)`였음.
	- 여기서 만든 **`Theme`들이 이벤트임**
![[Pasted image 20260508163132.png]]

3. 제일 큰 화면에서 `Add Timeline Sheet`으로 타임라인 시트를 추가한 다음, 그 안에서 다시 `Add Audio Track`으로 오디오 트랙이 들어갈 공간을 추가한다. 추가된 오디오 트랙에 오디오 파일을 드래그 & 드랍한다.
	- 추가로, 생성된 이벤트에 `Assign To Bank > Master`로 뱅크에 할당해준다. 할당되어 있지 않다면 위 이미지처럼 `#unassigned`가 나타남.

![[Pasted image 20260508171638.png]]
> 개념 정리 1. 
> 위 화면이 `Event Editor`이다.
> - **여기서 만드는 폴더(`BGM, SFX`) 자체는 아무 기능이 없다**. 파일 시스템의 폴더랑 동일함.
> - **각 이벤트는 기본적으로는 1개의 음원을 사용한다는 느낌으로 접근**한다. 
> 	- 내 경우, `BossStageTheme`은 2개의 음원을 사용하되 보스 스폰 여부에 따라 한쪽을 뮤트하고 다른 쪽을 켜는 방식이기 때문에 2개의 음원이 들어가 있음.

![[Pasted image 20260508171943.png]]
> 개념 정리 2.
> 위 화면은 `Mixer`이다.
> - `Event Editor`에서 만든 이벤트는 기본적으로 어떠한 그룹에도 포함되지 않는다. `Mixer`에서 각 이벤트에 연결될 그룹을 설정해준다.
> 	- 이미 그룹에 포함된 이벤트를 `Ctrl + D`로 복사해서 생성하면, 복사된 이벤트도 해당 그룹에 포함된 채로 생성된다.
> - 그래서 **전체적인 연결 방식은 `Event의 음원 -> Event의 마스터 트랙 -> Event가 포함된 Group의 믹서 버스 -> 전체 Master Bus`** 가 된다.

4. 빌드하기 : `F7` 
---
## 유니티에서의 과정
- `FMOD for Unity` 패키지 설치했다는 전제 하에 진행함

### 1. 상단 FMOD > Edit Settings
![[Pasted image 20260508174344.png]]
- `Single Platform Build`로 설정, `Build Path`도 위 `FMOD`에서 설정한 빌드 결과물이 있는 폴더로 설정한다.

> 처음엔 `FMOD Studio Project`를 선택했으나, 이상한 경로를 찾고 있다. 빌드 경로가 아니었는데 `내 문서`로 시작되는 곳을 찾고 있다. 지정된 경로와 다른 곳을 찾기 때문에 직관적이지 않고 이해되지 않았다. 조금 더 이해가 가는 방식인 이쪽으로 진행
---
### 2. 간단한 스크립트 작성 후 실행 테스트
```cs
using UnityEngine;
using FMODUnity;

public class FMODTest : MonoBehaviour
{
    void Start()
    {
        RuntimeManager.PlayOneShot("event:/BGM/LobbyTheme");
    }
}
```
- 인게임 매니저 오브젝트에 붙여서 실행 -> 잘 동작함

>[!note]
>`RuntimeManager`
>- FMOD의 핵심 관리자 클래스.
>- 유니티의 씬과 무관하게, 게임 전체 생명주기 동안 FMOD 시스템을 유지하는 싱글톤.
>- 기능
>	- FMOD 엔진 초기화 / 관리 
>	- 뱅크 로드 / 언로드
>	- 이벤트 생성 / 재생
>	- 3D 사운드 위치 업데이트

---
### 3. 새로운 SoundManager 생성
- 기존 `SoundManager`는 `SoundManager_Legacy`라는 이름으로 스크립트만 따로 저장

#### 볼륨 조절 기능 구현 - VCA (FMOD Studio)
- **`VCA`라는 게 필요하다고 한다.** 
	- `Volume Control Automation`이라는 듯
	- `Group`은 실제로 오디오가 흐르는 통로
	- `VCA`는 오디오가 얼마나 흐를지(=볼륨)만 결정하는 요소라고 보면 됨
- ~~사실 `Bus`를 직접 제어해도 되긴 하지만 용어가 나온 김에 적용까지 해보자. 밥 먹고.~~


- `Mixer` 탭을 보면
![[Pasted image 20260508203200.png]]
- `VCAs`라고 되어 있는 탭이 있다. 여기서 3개의 VCA를 만들어준다. 
	- `Master`
	- `BGM`
	- `SFX`


- 마스터 버스 / 그룹 의 연결은 `Routing` 탭에서 진행한다. 즉, `Routing`에서 `VCA`에 연결하는 개념이다. `VCA`에서는 어떤 그룹에 연결하는지 지정할 수 없음
	- `Master Bus, BGM Group, SFX Group` 각각을 우클릭해서 `Assign to VCA`로 해당되는 `VCA`를 연결함

- **`FMOD Studio`에서 변경 사항이 있다면 `F7`로 꼭 빌드해주자.** 
#### SoundManager 구현
```cs
using UnityEngine;
using FMODUnity;
using FMOD.Studio;

public class SoundManager : MonoBehaviour
{
    // FMOD.Studio.VCA - 볼륨 리모컨 역할
    private VCA vcaMaster;
    private VCA vcaBGM;
    private VCA vcaSFX;

    private void Awake()
    {
        InitVCAs();
        LoadVolume();
    }

    private void InitVCAs()
    {
        vcaMaster = RuntimeManager.GetVCA("vca:/Master");
        vcaBGM = RuntimeManager.GetVCA("vca:/BGM");
        vcaSFX = RuntimeManager.GetVCA("vca:/SFX");
    }

    // 1회 실행, PlayerPrefs에 저장된 값을 현재 볼륨으로 적용
    private void LoadVolume()
    {
        SetMasterVolume(PlayerPrefs.GetFloat("MasterVolume", 1f));
        SetBGMVolume(PlayerPrefs.GetFloat("BGMVolume", 1f));
        SetSFXVolume(PlayerPrefs.GetFloat("SFXVolume", 1f));
    }

    private void Start()
    {

    }

    // UI에서 필요할 때 호출, VCA 현재값 반환
    public float GetMasterVolume()
    {
        vcaMaster.getVolume(out float volume);
        return volume;
    }
    public float GetBGMVolume()
    {
        vcaBGM.getVolume(out float volume);
        return volume;
    }
    public float GetSFXVolume()
    {
        vcaSFX.getVolume(out float volume);
        return volume;
    }

    public void SetMasterVolume(float volume)
    {
        vcaMaster.setVolume(volume);
        PlayerPrefs.SetFloat("MasterVolume", volume);
    }
    public void SetBGMVolume(float volume)
    {
        vcaBGM.setVolume(volume);
        PlayerPrefs.SetFloat("BGMVolume", volume);
    }
    public void SetSFXVolume(float volume)
    {
        vcaSFX.setVolume(volume);
        PlayerPrefs.SetFloat("SFXVolume", volume);
    }
}
```

- **아래 로직의 반복임**
```cs
private VCA vcaMaster;

// vca 지정하기
vcaMaster = RuntimeManager.GetVCA("vca:/Master");

// vca의 볼륨값 가져오기 
public float GetMasterVolume()
{
	// FMOD API에서 볼륨값 얻기는 out 형식으로 제공됨
	vcaMaster.getVolume(out float volume);
	return volume;
}

// vca의 볼륨값 설정하기
public void SetMasterVolume(float volume)
{
	vcaMaster.setVolume(volume);
	PlayerPrefs.SetFloat("MasterVolume", volume);
}
```
> 이외의 `PlayerPrefs`로 값을 저장하고 가져오는 방식은 동일하다. 

---
