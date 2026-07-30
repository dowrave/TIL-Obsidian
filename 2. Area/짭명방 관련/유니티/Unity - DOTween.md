#Unity 

ToC

1. [[#DoTween.Sequence()|DoTween.Sequence()]]
	1. [[#DoTween.Sequence()#추가 옵션|추가 옵션]]
2. [[#DOAnchorPosX(endValue, duration)|DOAnchorPosX(endValue, duration)]]
3. [[#SetEase|SetEase]]




- 애니메이션을 만들어주는 라이브러리.

## DoTween.Sequence()
```cs
Sequence animationSequence = DOTween.Sequence()
```
> 애니메이션들이 들어갈 하나의 시퀀스를 정의함. 
> 개인적으로는 케라스의 모델 함수형 정의랑 비슷한 것 같기도? 마지막에 이어붙이는 개념은 아니지만.

### 추가 옵션
- `SetUpdate(bool isIndependentUpdate)` : `Time.TimeScale`과 별도로 동작시키느냐 여부. `true`일 때 유니티의 시간과 상관 없이 별도로 흘러간다. 
	- 게임이 멈춘 상황에서 애니메이션을 실행시키고 싶다면 `true`로 해줘야 함
- `SetAutoKill()` : 시퀀스가 완료된 후 자동으로 정리해준다.
## DOAnchorPosX(endValue, duration)
- `RectTransform`의 `anchoredPosition.x` 값을 현위치에서 `endValue`까지 `duration` 시간 동안 변화시킴

## SetEase
- 애니메이션의 뒤에 붙으며, 해당 위치 변화가 어떤 속도 곡선을 따를지를 정한다.

- 주요 이징 효과
	- `Linear` : 일정 속도
	- `InXXX` : 시작이 느리고 끝이 빠름(가속)
	- `OutXXX` : 시작이 빠르고 끝이 느림(감속)
	- `InOutXXX` : 시작, 끝이 느리고 중간이 빠름

- 강도 순서(약 -> 강)
- `Sine` : 가장 부드러움
- `Quad`
- `Cubic`
- `Quart`
- `Quint`
- `Expo` : 지수 곡선
- `Circ` : 원형 곡선
- `Back` : 약간 뒤로 갔다 오는 효과
- `Elastic` : 탄성 효과
- `Bounce` : 통통 튀는 효과


- 예제
```cs
animationSequence.Append(textContainer.DOAnchorPosX(0, textSlideDuration) .SetEase(Ease.OutQuint));

animationSequence.Append(textContainer.DOAnchorPosX(-textOffset, textSlideDuration) .SetEase(Ease.InQuad)); // 가속도를 붙여 퇴장
```

## Append와 Join의 차이
- 시퀀스를 구성할 때, `Append`와 `Join`이라는 메서드가 있다.
```cs
Sequence sequence = DOTween.Sequence();
// 1초동안 페이드 인 시작
sequence.Append(canvasGroup.DOFade(1f, 1f));  
// 페이드 인과 동시에 1초동안 이동 시작
sequence.Join(transform.DOMoveX(5f, 1f));
```
> 설명에서 보이듯, 
> **- `Append`은 이전 애니메이션이 종료된 후에 실행된다.**
> **- `Join`은 이전 애니메이션과 동시에 실행된다.**


