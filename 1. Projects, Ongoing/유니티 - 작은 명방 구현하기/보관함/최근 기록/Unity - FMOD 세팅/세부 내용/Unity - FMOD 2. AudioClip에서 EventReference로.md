#유니티-FMOD 
## 파일 선택 및 재생(AudioClip -> EventReference)
- 기존 오디오는 `AudioClip` 타입을 이용했음
- 관리하는 곳은 여러 `SO`에 나뉘었다.
	- 전체 공용이라면 `SoundDatabase`
	- 유닛마다 지정할 요소는 유닛이나 스킬의 `SO`
	- 스테이지마다 지정할 요소도 스테이지의 `SO`

- **`AudioClip` 타입을 `FMODUnity.EventRefernce`로 변경해줌**
![[Pasted image 20260508210645.png]]
> 그러면 인스펙터가 이렇게 바뀐다. 돋보기 클릭해서 지정하고 싶은 요소를 지정하면 됨.

>[!warning]
>- 여기서 이슈 발생 
>![[Pasted image 20260508211610.png]]
>- 위 항목을 보면 `OneShot : true`로 되어 있음 : 즉 1번만 실행된다는 의미임
>- `FMOD Studio`에서 해당 `Event`의 `Loop`를 켰는데도 반영이 안된다. 빌드를 다시 해봐도 반영이 안됨.

---
### FMOD Studio에서 이벤트에 루프 세팅하는 방법
![[Pasted image 20260508212344.png]]
> 음원이 들어간 **오디오 트랙 우클릭 > `New Loop Region`로 설정** 
![[Pasted image 20260508212503.png]]

- 참고) `FMOD Studio`에서 상단의 루프 표시나 오디오 트랙 클릭했을 때 하단에 나타나는 박스의 루프 표시 모두 효과 없음. `New Loop Region`만 효과가 있었다.

---
### Stream 관련
![[Pasted image 20260508214346.png]]
- `Event Editor`의 좌측 탭의 `Assets`으로 접근한다. 


![[Pasted image 20260508214441.png]] ![[Pasted image 20260508214602.png]]
- 각 요소를 클릭하면 위 2가지 중 하나로 나타난다. 
	- 좌측은 `Advanced Loading Mode`가 나타나는데, 여기서 `Compressed/Decompressed/Streaming` 중 1가지로 설정할 수 있음.
	- 우측은 단순히 `Loading`의 `STREAM`을 활성화/비활성화 처리.

- `Assets`을 관리하는 것에서 보이듯, 음원 단위로 처리된다.
	- 정확히는 이벤트 안의 음원 중 1개라도 `Streaming`이 켜져 있다면 유니티에서 `true`, 아니라면 `false`로 나타남.

> 변경 후 빌드했을 때 유니티에서 `Stream` 값이 바뀌는지 여부까지 확인했음
#### 기능 설명
- 말 그대로 스트리밍이다. 음원을 잘라서 그때그때 올리냐 한꺼번에 올리냐 차이.
- 요소에 따른 설정법
	- `BGM : true`
		- 일반적으로 길기 때문에 메모리에 한꺼번에 올리면 부담되기 때문(I/O가 일반적으로 제일 느리다는 걸 생각하면 됨)
	- `SFX : false`
		- 짧기 때문에 메모리에 올려두는 게 유리함

---

> 추가 참고
> - `AudioClip`은 `null`비교가 가능하다 (`!= null`)
> - `EventReference`는 구조체라서 `null` 비교가 불가능하다. `IsNull` 프로퍼티로 접근해야 함.

---

## Sting, SFX 처리
![[Pasted image 20260511164246.png]]
> - 유니티에 있던 BGM, SFX, Sting들 다 옮기고 `Assign to Bank > Browse > Master`로 설정
> - `Sting`은 BGM처럼 처리되는 것도 있고 SFX처럼 처리되는 것도 있어서 뒤에 이름 더 붙여둠


---
## AudioClip => EventReference 변경 과정 중 이슈

1. 파라미터로 전달하던 `AudioClip? field = null`은 `EventReference`로 바꾸면 `nullable`이 아니기 때문에 아예 형태가 달라져야 함
```cs
// before
public AttackSource(AudioClip hitSFX = null)
{
	HitSFX = hitSFX;
}

// after : default로 설정
public AttackSource(EventReference hitSFX = default)
{
	HitSFX = hitSFX;
}
```

>[!note]
>- `default`란?
>- **`default(struct)`는 모든 필드를 각 타입의 기본값으로 초기화한다.** 
>- `FMODUnity.EventReference`는 `GUID`를 비롯한 필드들을 가졌으며, `default`로 달아뒀을 경우 `IsNull = true`가 됨.

- `AudioClip`으로 설정됐던 요소들 모두 `EventReference` 기반으로 바꾸는 중...
- `VS Code` 기준, **`Shift + F`로 검색 기능을 여러 파일에 대해 일괄적으로 수행**할 수 있다. 처음 알았다. 왜 그 고생을 했나 싶어지는..

---

## FMOD에서의 BGM 관리 - EventInstance
- `RuntimeManager.PlayOneShot`는 여기서도 유니티 내장 오디오처럼 "쏘고 잊는" 방식임 -> 그래서 BGM 용도는 아님

- BGM은 `EventInstance`라는 타입을 활용한다. 
	- `EventReference`랑은 구분할 것.
	- 루프 설정은 `FMOD` 자체의 이벤트 단위로 관리한다. 

- 유니티에선 `AudioSource`에 `clip`만 바꾸는 식으로 구현했다면, 여기선 **개별 BGM은 개별 `EventInstance`를 쓴다**는 전제로 작업하면 됨.

### 관련 메서드들
- 메서드의 이름들은 모두 소문자로 시작함에 유의. 
	- 이런 걸 `카멜 케이스`라고 함. (대문자로 시작하면 `파스칼 케이스`)

- **`isValid()` - `EventInstance`의 `null` 체크**
	- 포인터를 래핑한 구조체다. 
	- `IsNull`이 아니라 **`isValid()`** 라는 메서드를 쓴다.
	- 포인터가 가리키는 게 있다면 `true`, 없다면 `false`이다. 
	- `default(EventInstance)`는 `IntPtr.Zero`라서 `isValid()`가 `false`를 반환한다.

- `stop()` - 곡을 멈추고 초기화함
	- 모드 
		- `FMOD.Studio.STOP_MODE.ALLOWFADEOUT` - 페이드아웃
		- `FMOD.Studio.STOP_MODE.IMMEDIATE` - 바로 멈추기
- `SetPaused(bool pause)` - 일시정지, `true`일 때 멈춤

---
