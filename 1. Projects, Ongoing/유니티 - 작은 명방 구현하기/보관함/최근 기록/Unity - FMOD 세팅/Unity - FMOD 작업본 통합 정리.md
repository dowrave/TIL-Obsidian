
- `Claude`로 `260508 ~ 260515` 일자별 내용 넘겨주고 요약함

---
## 목차
1. [[#기본 개념 및 구조]]
2. [[#FMOD Studio 세팅]]
3. [[#Unity 연결 세팅]]
4. [[#SoundManager 구현]]
    - [[#VCA - 볼륨 조절]]
    - [[#EventInstance - BGM 관리]]
    - [[#BGM 전환 로직 (Parameter)]]
5. [[#Snapshot - 덕킹 및 LPF 효과]]
6. [[#기타 설정 및 트러블슈팅]]
    - [[#Loop 세팅]]
    - [[#Stream 설정]]
    - [[#AudioClip → EventReference 변경]]
    - [[#3D 사운드 및 Spatializer]]
    - [[#동시 재생 수 제한 (Max Instances)]]

---

## 기본 개념 및 구조

### Unity 기본 오디오 vs FMOD 비교

|Unity 기본 오디오|FMOD|
|---|---|
|`mp3, wav` 파일|이벤트(Event)|
|`AudioClip`을 `AudioSource`에 붙임|이벤트를 뱅크에 담아서 빌드|
|파일 그대로 사용|`.bank` 파일로 패키징해서 사용|

### 핵심 용어

- **이벤트(Event)** : 개별 사운드 요소
- **뱅크(Bank)** : 개별 이벤트들을 유니티가 읽을 수 있게 압축/포장한 파일. 특정 뱅크는 특정 씬에서만 로드되도록 구성할 수 있다.
- **VCA** : 전압 제어 증폭기(Voltage Controlled Amplifier). 오디오가 얼마나 흐를지(=볼륨)만 결정하는 요소. Group은 실제로 오디오가 흐르는 통로이고, VCA는 그 양을 제어하는 리모컨 역할.
- **Snapshot** : 믹서에 일시적으로 씌우는 레이어. **FMOD는 기본 믹서 상태를 항상 별도로 유지**하므로, 스냅샷이 해제되면 원래 세팅으로 자동 복구된다.
- **EventInstance** : BGM처럼 상태 관리가 필요한 오디오를 제어하는 타입. `EventReference`와 구분할 것.

### 전체 신호 흐름

```
Event의 음원 → Event의 마스터 트랙 → Event가 포함된 Group의 믹서 버스 → 전체 Master Bus
```

### RuntimeManager

- FMOD의 핵심 관리자 클래스
- 유니티의 씬과 무관하게, 게임 전체 생명주기 동안 FMOD 시스템을 유지하는 싱글톤
- 기능 : FMOD 엔진 초기화/관리, 뱅크 로드/언로드, 이벤트 생성/재생, 3D 사운드 위치 업데이트

---

## FMOD Studio 세팅

### 1. 필요한 것

- **패키지** : `FMOD for Unity` (유니티 패키지 매니저)
- **스튜디오** : `FMOD Studio` (웹에서 별도 설치)
- 스튜디오와 패키지의 버전은 일치시키는 게 좋다.

### 2. 프로젝트 생성

- `FMOD Studio`에서 새로운 프로젝트 생성
- 프로젝트는 유니티 프로젝트 외부에 저장

### 3. 뱅크 빌드 경로 설정

1. `Edit > Preferences > Build > Built banks output directory`를 유니티 프로젝트의 `/Assets/StreamingAssets/fmod`로 설정
    - `StreamingAssets/`, `fmod/`는 수동으로 만들어두자.
2. `F7`로 빌드 → `Master.bank`, `Master.strings.bank` 생성 확인

> **FMOD Studio에서 변경 사항이 있다면 `F7`로 꼭 빌드해주자.**

### 4. Mixer 구성 (Routing)

- 왼쪽 `Routing - Add Group`으로 `BGM`, `SFX` 그룹 추가
- 기존 Unity Audio Mixer Group(`Master, BGM, SFX`)과 동일한 구조로 맞춤

### 5. Event Editor 구성

- `Window > Event Editor(Ctrl + 1)`으로 탭 전환
- `BGM`, `SFX` 폴더 생성 후 폴더 안에 `New Event`로 이벤트 추가
    - **폴더 자체는 아무 기능이 없다**. 파일 시스템의 폴더와 동일.
    - **각 이벤트는 기본적으로는 1개의 음원을 사용한다는 느낌으로 접근**

```
[Event Editor]
├── BGM/
│   ├── LobbyTheme
│   └── StageTheme (보스 트랙 2개 포함)
└── SFX/
    └── ...
```

- 이벤트 생성 후 : `Add Timeline Sheet` → `Add Audio Track` → 오디오 파일 드래그&드랍
- 이벤트를 뱅크에 할당 : `Assign To Bank > Master`
    - 할당되지 않은 이벤트는 `#unassigned`로 표시됨

### 6. Mixer 탭에서 Event의 그룹 연결

- Event Editor에서 만든 이벤트는 기본적으로 어떤 그룹에도 포함되지 않는다.
- `Mixer(Ctrl + 2)` 탭에서 각 이벤트에 연결될 그룹을 설정
- `Ctrl + D`로 이벤트를 복사하면, 복사된 이벤트도 같은 그룹에 포함된 채로 생성된다.

---

## Unity 연결 세팅

### FMOD > Edit Settings

- `Single Platform Build`로 설정
- `Build Path`를 FMOD Studio에서 빌드한 결과물이 있는 폴더로 설정

> `FMOD Studio Project` 옵션은 지정된 경로와 다른 경로(내 문서 등)를 찾는 경우가 있어서 `Single Platform Build`가 더 직관적이다.

### 기본 재생 테스트

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

---

## SoundManager 구현

### VCA - 볼륨 조절

#### FMOD Studio 설정

- `Mixer` 탭의 `VCAs` 항목에서 `Master`, `BGM`, `SFX` VCA 3개 생성
- `Routing` 탭에서 `Master Bus`, `BGM Group`, `SFX Group` 각각을 우클릭 → `Assign to VCA`로 연결
    - VCA에서는 어떤 그룹에 연결할지 지정할 수 없다. Routing에서 연결하는 구조.

#### Unity 코드

```cs
using UnityEngine;
using FMODUnity;
using FMOD.Studio;

public class SoundManager : MonoBehaviour
{
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
        vcaBGM    = RuntimeManager.GetVCA("vca:/BGM");
        vcaSFX    = RuntimeManager.GetVCA("vca:/SFX");
    }

    // PlayerPrefs에 저장된 값으로 볼륨 복원
    private void LoadVolume()
    {
        SetMasterVolume(PlayerPrefs.GetFloat("MasterVolume", 1f));
        SetBGMVolume(PlayerPrefs.GetFloat("BGMVolume", 1f));
        SetSFXVolume(PlayerPrefs.GetFloat("SFXVolume", 1f));
    }

    // UI에서 현재 값을 가져올 때 사용
    public float GetMasterVolume() { vcaMaster.getVolume(out float v); return v; }
    public float GetBGMVolume()    { vcaBGM.getVolume(out float v);    return v; }
    public float GetSFXVolume()    { vcaSFX.getVolume(out float v);    return v; }

    // FMOD API에서 볼륨값 얻기는 out 형식으로 제공됨
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

---

### EventInstance - BGM 관리

- `RuntimeManager.PlayOneShot`은 "쏘고 잊는" 방식 → BGM 용도로는 부적합
- BGM은 `EventInstance` 타입을 활용
- 유니티의 `AudioSource`에 `clip`만 바꾸는 방식과 달리, **개별 BGM은 개별 `EventInstance`**를 사용한다는 전제로 접근

#### 주요 메서드

> 메서드 이름이 모두 소문자(카멜 케이스)로 시작하는 것에 유의

- **`isValid()`** : `EventInstance`의 null 체크. 포인터를 래핑한 구조체이므로 `IsNull`이 아닌 이 메서드를 사용. `default(EventInstance)`는 `IntPtr.Zero`라서 `false`를 반환.
- **`stop(mode)`** : 재생 중지
    - `FMOD.Studio.STOP_MODE.ALLOWFADEOUT` - 페이드아웃
    - `FMOD.Studio.STOP_MODE.IMMEDIATE` - 즉시 중지
- **`setPaused(bool)`** : 일시정지. `true`일 때 정지.

---

### BGM 전환 로직 (Parameter)

기존 유니티 오디오 방식은 2개의 AudioSource를 동시에 틀어놓고 한쪽을 뮤트하는 방식이었다면, FMOD에서는 **하나의 이벤트에 2개의 트랙을 넣고 파라미터로 볼륨 오토메이션을 제어**하는 방식으로 구현한다.

#### FMOD Studio 설정

1. 하나의 이벤트에 2개의 오디오 트랙을 추가. 기본 상태에서 보스 BGM 트랙은 뮤트.
2. `Create > Add Parameter(Ctrl + P)`로 파라미터 생성 (`Continuous, 0 ~ 1, Local`)
3. 각 트랙의 볼륨 게이지 우클릭 → `Add Automation`으로 오토메이션 생성
4. 우측 `Overview`의 파라미터 항목 체크박스 클릭 (또는 `Timeline` 우측 탭 → `Add Parameter Sheet`)
5. 파라미터 시트에서 볼륨 오토메이션 커브 설정
    - 예: `0 ~ 0.5` : 기존 BGM 페이드아웃 / `0.5 ~ 1` : 보스 BGM 페이드인

#### Unity 코드

```cs
// SoundManager.cs

// BGM 재생
currentBGM = RuntimeManager.CreateInstance(bgmRef);
currentBGM.setParameterByName(BGMTransitionParam, 0f); // 파라미터 없는 이벤트에도 오류 안 남
currentBGM.start();

// BGM 전환
public void SwitchTrack()
{
    if (_isSwitching) return;
    StartCoroutine(MoveParameter(from: 0f, to: 1f, transitionDuration));
}

private IEnumerator MoveParameter(float from, float to, float duration)
{
    _isSwitching = true;

    float elapsed = 0f;
    while (elapsed < duration)
    {
        elapsed += Time.unscaledDeltaTime;
        float t = Mathf.Clamp01(elapsed / duration);
        currentBGM.setParameterByName(BGMTransitionParam, Mathf.Lerp(from, to, t));
        yield return null;
    }

    currentBGM.setParameterByName(BGMTransitionParam, 1f);
    _isSwitching = false;
}
```

> `setParameterByName`은 파라미터가 없는 이벤트에도 오류를 일으키지 않는다.

---

## Snapshot - 덕킹 및 LPF 효과

### 개념

- **스냅샷** : "어떤 상태로 만들라"는 명령을 담은 프리셋. 믹서에 일시적으로 씌우는 레이어.
	- FMOD는 기본 믹서 상태를 별도로 유지하므로, **스냅샷 해제 시 자동 복구**. 원래 값을 코드로 기억해둘 필요 없음.
	- 복구에 걸리는 시간은 `AHDSR` 설정 시 `Release` 값을 따름
- **Scope** : 스냅샷이 영향을 주는 프로퍼티 범위. 스튜디오에서 건드린 값은 자동으로 Scope In됨(붉은색 표시).
- **Blending** : 볼륨 프로퍼티만 더하는 방식, 나머지는 덮어씌움.

### 덕킹(Ducking)

- 어떤 오디오 신호가 나올 때 다른 오디오 신호를 줄이는 방식 (사이드 체인과 유사한 개념)
    - 차이점 : 사이드 체인은 기준 신호의 영향을 받아 감쇄량이 변하지만, 덕킹은 개발자가 감쇄량을 고정으로 정하는 방식

### 페이드 처리 - AHDSR Modulator

FMOD 권장 방식. `Intensity`에 `AHDSR Modulator`를 적용해 페이드를 구현한다.

- `Intensity` : 설정한 값을 얼마나 적용할지의 비율
    - 예: `-15dB`로 설정 + `Intensity 50%` → 실제로는 `-7.5dB` 적용
    - 일반적으로는 목표값을 1번 설정에서 다 잡아두므로 Intensity는 건드릴 일이 잘 없음
- 코드에서는 `start/stop`만 호출하면 되므로 간단함

#### FMOD Studio 설정 순서

1. 스냅샷에서 원하는 최종값을 Scope로 잡아둠
2. 하단 `Intensity → Add Modulation → AHDSR` 설정
3. `Event Editor`에서 스냅샷 이벤트를 실행시켜보며 테스트

### 실제 구현 - BossSkillDucking 스냅샷

#### FMOD Studio

- `Mixer`에서 `BossSkillDucking` 스냅샷 생성
- VCA로 `BGM`, `SFX` 값을 `-40dB`로 낮춤 (테스트하며 조정)

#### Unity 코드

```cs
EventInstance dropSFXInstance = default;
EventInstance duckingSnapshot = default;

// 낙하 SFX + 스냅샷 시작
if (!_skill.ParticleDropSFX.IsNull)
{
    dropSFXInstance = RuntimeManager.CreateInstance(_skill.ParticleDropSFX);
    dropSFXInstance.start();

    duckingSnapshot = RuntimeManager.CreateInstance("snapshot:/BossSkillDucking");
    duckingSnapshot.start();
}

yield return new WaitForSeconds(skillData.FallDuration);

PlayExplosionVFX();
ApplyInitialEffect(null);

// 낙하 SFX 즉시 중지
if (dropSFXInstance.isValid())
{
    dropSFXInstance.stop(FMOD.Studio.STOP_MODE.IMMEDIATE);
    dropSFXInstance.release();
}

// 폭발 SFX 재생 + 스냅샷 페이드아웃 복구
if (!_skill.ExplosionSFX.IsNull)
{
    RuntimeManager.PlayOneShot(_skill.ExplosionSFX);

    duckingSnapshot.stop(FMOD.Studio.STOP_MODE.ALLOWFADEOUT); // 서서히 복구
    duckingSnapshot.release();
}
```

> FMOD의 타임 스케일은 유니티와 별도로 동작. 복구 속도는 AHDSR의 `Release` 값으로 결정.

### LPF (Low Pass Filter) - 먹먹한 효과

볼륨만 낮추는 것 외에, 고음을 깎아서 소리를 먹먹하게 만드는 효과. 볼륨 감쇄 + LPF를 함께 적용하면 볼륨을 덜 깎아도 효과가 충분해진다.

#### FMOD Studio 설정

1. `Routing`의 각 그룹(`BGM`, `SFX`)의 `Master Track` 좌측 → `Add Effect > FMOD Deprecated > FMOD Low Pass` 추가
    
2. 디폴트 상태에서 `Cutoff`를 최댓값으로 설정 (어떤 주파수도 필터링하지 않음)
    
3. `Snapshots` 탭에서 원하는 스냅샷 클릭 → 믹서가 스냅샷 설정 창으로 전환됨
    
    > 현재 설정이 기본인지 스냅샷인지 헷갈릴 수 있음. **스냅샷 탭이 선택된 상태면 스냅샷 설정**을 하고 있는 것.
    
4. 스냅샷 설정 창에서 `Routing`의 그룹 클릭 → Scope에 잡히지 않은 값은 점선으로 표시됨
    
5. `Cutoff` 값을 원하는 수치로 수정 → 스냅샷 활성화 시 해당 주파수 이상 필터링
    

---

## 기타 설정 및 트러블슈팅

### Loop 세팅

FMOD Studio에서 루프를 설정하는 올바른 방법 :

- **오디오 트랙 우클릭 → `New Loop Region`으로 설정**

> 상단의 루프 표시나 오디오 트랙 클릭 시 하단에 나타나는 박스의 루프 표시는 효과 없음. `New Loop Region`만 동작함.

---

### Stream 설정

`Event Editor` 좌측 탭의 `Assets`에서 음원 단위로 설정.

- **BGM : Stream = true**
    - 일반적으로 길어서 메모리에 한꺼번에 올리면 부담됨
- **SFX : Stream = false**
    - 짧아서 메모리에 올려두는 게 재생 지연 없이 유리함

> 이벤트 안의 음원 중 1개라도 Streaming이 켜져 있으면 유니티에서 `Stream: true`로 표시됨.

---

### AudioClip → EventReference 변경

- **`AudioClip` 타입을 `FMODUnity.EventReference`로 변경**
- `EventReference`는 구조체이므로 `null` 비교 불가 → **`IsNull` 프로퍼티**로 접근

```cs
// AudioClip은 null 비교 가능
if (audioClip != null) { ... }

// EventReference는 구조체라 null 비교 불가
if (!eventRef.IsNull) { ... }
```

- 파라미터 기본값도 변경 필요

```cs
// before
public AttackSource(AudioClip hitSFX = null) { ... }

// after : default로 설정 (default(EventReference) → IsNull = true)
public AttackSource(EventReference hitSFX = default) { ... }
```

> VS Code에서 `Shift + F`로 여러 파일에 대해 일괄 검색/치환 가능.

---

### 3D 사운드 및 Spatializer

- **3D 이벤트 여부 기준** : 이벤트의 마스터 트랙에 `Spatializer`가 있는지 여부
- `CreateInstance` 후 `start()`만 하면 3D 이벤트는 소리가 나지 않음 → `set3DAttributes` 설정이 필요

```cs
// 3D 이벤트인 경우
dropSFXInstance = RuntimeManager.CreateInstance(_skill.ParticleDropSFX);
dropSFXInstance.set3DAttributes(RuntimeUtils.To3DAttributes(gameObject)); // 없으면 소리 안 남
dropSFXInstance.start();
```

- 2D 타임라인으로 생성된 이벤트는 `set3DAttributes` 불필요
- 프로젝트에서 3D 사운드가 필요 없다면 모든 이벤트의 `Master Track`에서 `Spatializer`를 제거하면 된다.

---

### 동시 재생 수 제한 (Max Instances)

같은 SFX가 너무 많이 동시에 재생되는 경우 스튜디오 레벨에서 제한 가능.

- `Event Editor`에서 이벤트 선택 → 우측 하단 `Event Macros` → `Max Instances` 수정
- 디폴트는 무한
- 예: 문제가 됐던 `LightningHit` → `Max Instances = 2`로 설정

> `Polyphony` 설정도 있으나, 동시 인스턴스 수 제한에는 `Max Instances`가 더 직접적으로 관련된 설정이다.

---

## 참고 - 단축키 모음

|단축키|기능|
|---|---|
|`F7`|뱅크 빌드|
|`Ctrl + 1`|Event Editor|
|`Ctrl + 2`|Mixer|
|`Ctrl + P`|파라미터 추가|
|`Ctrl + D`|이벤트 복사|
