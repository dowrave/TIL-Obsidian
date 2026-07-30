## 사운드 매니저, 사운드 DB 추가
- 소리에 대해선 아예 아는 게 없기 때문에 관련 지식부터 쌓고 시작함
- 공부는 제미나이랑 클로드한테 똑같이 물어보고 답변 받음

### 유니티 소리 재생 방식
- 3가지 구성 요소가 있음.
1. `Audio Listener(귀)` : 씬에 단 하나만 존재해야 한다. `Main Camera`에 붙어 있다.
2. `Audio Soruce(스피커)` : 소리를 내보낸다. 게임 내 오브젝트에 부착되어 소리를 발생시킨다.
3. `Audio Clip(음원)` : 실제 소리 데이터인 음원 파일`.mp3, .wav. .ogg`이다. 

### 소리 실행 방식
- 보통 1개의 BGM / 효과음은 여러 개가 동시에 재생되는 구조
- 같은 소리의 중복 재생? 

- 효과음 찾는다고 시간 잡아먹었다. 이거 찾는 것도 일이겠음;
### 구현
- 크게 2단계로 나뉜다. 

1. 오디오 클립들을 관리할 DB 역할을 하는 SO
2. 이를 실행할 `SoundManager`
	- 내 경우 `GameManagement`의 자식 오브젝트로 넣었음

#### SoundDatabase
- `AudioClip`들을 갖고 있는 역할
```cs
using UnityEngine;

[CreateAssetMenu(fileName = "SoundDatabase", menuName = "Audio/Sound Database")]
public class SoundDatabase : ScriptableObject
{
    [Header("UI Sounds")]
    [SerializeField] private AudioClip buttonClick;

    [Header("BGM")]
    [SerializeField] private AudioClip mainMenuBGM;

    public AudioClip ButtonClick => buttonClick;
    public AudioClip MainMenuBGM => mainMenuBGM;
}
```
> 참고) **`SO`의 필드는 `public`으로 작성하는 방식이 훨씬 많이 쓰임**
> - 규모가 큰 프로젝트라면 위 방식을 택해서 오류 가능성을 줄이지만 작은 프로젝트라면 `public`으로만 쓰고 주석으로 할당하지 말라고 표시만 해도 무방함
> - 그냥 `public`으로 구현하고 주석으로 할당하지 말라고만 달아놔도 무방한 내용임. 지금까지의 패턴이 있으니까 난 유지하는 걸로.

- `[SerializeField] private` 패턴은 `MonoBehaviour`에서 많이 사용함 

#### SoundManager
```cs
using UnityEngine;

public class SoundManager : MonoBehaviour
{
    [Header("Sound DB")]
    [SerializeField] private SoundDatabase soundDB;

    [Header("Audio Sources")]
    [SerializeField] private AudioSource bgmSource;
    [SerializeField] private AudioSource[] sfxSources;

    [Header("SFX Pool Settings")]
    [SerializeField] private int sfxSourceCount = 5; // 동시에 실행 가능한 효과음 갯수

    [Header("Volume Settings")]
    [SerializeField][Range(0f, 1f)] private float bgmVolume = 0.7f;
    [SerializeField][Range(0f, 1f)] private float sfxVolume = 1f;

    private void OnEnable()
    {
        InitializeAudioSources();
    }

    // SoundManager에 AudioSource들을 만드는 과정
    private void InitializeAudioSources()
    {
        // bgm 관련 설정들
        if (bgmSource == null)
        {
            bgmSource = gameObject.AddComponent<AudioSource>();
            bgmSource.loop = true;
            bgmSource.playOnAwake = false;
        }

        // 효과음 관련 설정들
        sfxSources = new AudioSource[sfxSourceCount];
        for (int i = 0; i < sfxSourceCount; i++)
        {
            if (sfxSources[i] == null)
            {
                sfxSources[i] = gameObject.AddComponent<AudioSource>();
            }

            sfxSources[i].volume = sfxVolume;
            sfxSources[i].playOnAwake = false;
        }

        bgmSource.volume = bgmVolume;
    }

    #region BGM Methods

    public void PlayBGM(AudioClip clip)
    {
        if (clip == null) return;

        // 이미 같은 BGM이 재생 중이면 메서드 실행 X 
        if (bgmSource.clip == clip && bgmSource.isPlaying) return; 

        bgmSource.clip = clip;
        bgmSource.Play();
    }

    public void StopBGM()
    {
        bgmSource.Stop();
    }

    public void SetBGMVolume(float volume)
    {
        bgmVolume = Mathf.Clamp01(volume);
        bgmSource.volume = bgmVolume;
    }

    #endregion
    #region BGM Methods

    public void PlaySFX(AudioClip clip)
    {
        if (clip == null) return;

        AudioSource availableSource = GetAvailableSFXSource();

        if (availableSource != null)
        {
            availableSource.PlayOneShot(clip);
        }
    }

    public void PlaySFX(AudioClip clip, float volumeScale)
    {
        if (clip == null) return;

        AudioSource availableSource = GetAvailableSFXSource();

        if (availableSource != null)
        {
            availableSource.PlayOneShot(clip, volumeScale);
        }
    }

    public void StopSFX()
    {
        foreach (AudioSource source in sfxSources)
        {
            source.Stop();
        }
    }

    public void SetSFXVolume(float volume)
    {
        sfxVolume = Mathf.Clamp01(volume);
        foreach (AudioSource source in sfxSources)
        {
            source.volume = sfxVolume;
        }
    }

    // 사용 가능한 AudioSource를 찾음.
    // 재생 중이지 않은 소스를 우선 반환하며, 모두 사용 중이라면 1번째 소스를 반환
    private AudioSource GetAvailableSFXSource()
    {
        // 재생 중이지 않은 오디오가 있다면 먼저 반환
        foreach (AudioSource source in sfxSources)
        {
            if (!source.isPlaying) return source;
        }

        // 재생 
        return sfxSources[0];
    }
    #endregion
    #region UI Sound Helper Methods

    public void PlayButtonClick()
    {
        PlaySFX(soundDB.ButtonClick);
    }

    #endregion
}
```
대충 이런 느낌으로 쓰더라~~ 정도만 캐치하면 되겠음.


버튼 여기저기에 할당했는데, `OnClick`에서 클릭 시 소리가 나도록 하는 유틸리티 클래스를 따로 구현했음
```cs
using UnityEngine;
using UnityEngine.UI;

/// <summary>
/// 버튼 사운드 관리 유틸리티 클래스
/// </summary>
public static class ButtonSoundUtility
{

    // 단일 버튼에 클릭 사운드 추가
    public static void AddClickSound(Button button)
    {
        if (button == null)
        {
            Debug.LogWarning("Button이 null입니다!");
            return;
        }
        
        // 중복 방지를 위해 기존 리스너 제거
        button.onClick.RemoveListener(PlayClickSound);
        button.onClick.AddListener(PlayClickSound);
    }
    

    // 여러 버튼에 클릭 사운드 추가
    public static void AddClickSound(params Button[] buttons)
    {
        foreach (Button button in buttons)
        {
            AddClickSound(button);
        }
    }
    
    // 버튼에서 클릭 사운드 제거
    public static void RemoveClickSound(Button button)
    {
        if (button == null) return;
        button.onClick.RemoveListener(PlayClickSound);
    }
    

    // 실제 사운드 재생 메서드
    private static void PlayClickSound()
    {
        if (GameManagement.Instance.Sound != null)
        {
            GameManagement.Instance.Sound.PlayButtonClick();
        }
    }
}

// AddClickSound(button); 으로 곧바로 소리 추가 가능
// 추가는 개별 코드에서 작업 - 프로젝트 자체가 살짝 뒤죽박죽인 감이 있어서 보면서 작업
```
