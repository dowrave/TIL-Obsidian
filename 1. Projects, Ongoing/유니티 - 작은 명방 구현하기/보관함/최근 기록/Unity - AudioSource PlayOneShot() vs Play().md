#Unity 

## Play()
- `AudioSource`가 하나의 클립을 소유하고 재생한다. 
- `Stop()`, `Pause()`, `isPlaying` 체크가 정상적으로 작동한다.
- `AudioSource.clip`에 `AudioClip`을 할당한 후 인자 없이 실행함
## PlayOneShot()
- `AudioSource`는 출력 채널로만 사용한다. 클립 재생 자체는 유니티 내부 오디오 엔진에 맡긴다. 재생 상태를 추적하지 않는다.
- "쏘고 잊는" 용도에 적합하다. 총소리, UI 클릭음처럼 재생 후 제어할 필요가 없는 짧은 효과음이다. 
- `source.PlayOneShot(clip)` 같이 인자를 전달해서 실행함


