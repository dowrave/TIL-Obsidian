- 폴더 : `Assets/Settings/Audio` 내에 `MyAudioMixer`을 만들어서 넣음
- 세팅

![[Pasted image 20260213162918.png]]
1. Groups을 클릭, `Master 아래에 BGM, SFX, UI가 오도록` 구조를 짬
	- `AudioSource`가 여기서 만든 개별 믹서와 연결되는 방식임.

2. 개별 믹서를 클릭하면 인스펙터에 
![[Pasted image 20260213164222.png]]
이렇게 나타난다. `Volume 글자 우클릭 -> Expose Volume To Script` 클릭.

3. 그러면 `Audio Mixer`의 우측 상단 `Exposed Parameters`의 값이 바뀐다.  
![[Pasted image 20260213164326.png]]
저걸 클릭해서 방금 만든 파라미터의 이름 부분을 더블 클릭(혹은 우클릭 - Rename)해서 이름을 바꿔준다. `Master`를 포함해서 총 4번 해줌

>[!note]
>- `AudioMixer` : 유니티에서 `Window > Audio > Audio Mixer`로 만든 애셋 파일
>- `AudioMixerGroup` : `AudioMixer`의 세부 그룹들. 1번에서 구성한 `Master, BGM, SFX, UI`가 해당된다.

4. 스크립트로 `AudioSource`에 어떤 믹서를 연결할지 설정한다.
```cs
[SerializeField] private AudioMixerGroup sfxGroup;
	
AudioSource source = gameObject.AddComponent<AudioSource>();
source.outputAudioMixerGroup = sfxGroup;
```

5. 볼륨 조절은 `AudioMixer`, 즉 애셋 파일에 접근해서 2번에서 설정한 파라미터로 접근한다.
```cs

[SerializeField] private AudioMixer masterMixer;

public void SetBGMVolume(float volume)
{
	bgmVolume = Mathf.Clamp01(volume);
	float dB = convert01TodB(bgmVolume); // 슬라이더의 0~1 값을 믹서에서 몇 데시벨로 깎을지 변환함
	masterMixer.SetFloat("BGM_Volume", dB); // 믹서에 반영
	PlayerPrefs.SetFloat("BGMVolume", volume); // 슬라이더값은 별도로 저장
}

// 0~1로 들어오는 인풋에 대해, dB가 얼마나 깎이는지로 변환해주는 로직
// -0dB(sliderValue = 1) ~ -80dB(sliderValue = 0)
private float convert01TodB(float sliderValue)
{
	float dB = Mathf.Log10(Mathf.Max(0.0001f, sliderValue)) * 20f;
	return dB;
}
```

