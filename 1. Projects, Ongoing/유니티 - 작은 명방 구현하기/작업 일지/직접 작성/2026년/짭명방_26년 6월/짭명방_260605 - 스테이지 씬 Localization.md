
>[!done]
>- 기타 이슈 / 오류 수정
>	- `StageScene`에서 배속이 느려진 경우, 속도 조절이 불가능하도록 변경
>		- 오퍼레이터 박스 / 배치된 오퍼레이터 클릭 시마다 상태 전환에 약간의 딜레이가 있는 문제 수정 및 리팩토링 진행



## 스테이지 씬 Localization 계속



## 기타 이슈 / 오류 수정
### `StageScene` : 배속이 느려진 상태에서는 (1 ~ 1.5배속) 속도 조절 불가능하도록 변경
1. 메서드에 조건 추가(단축키 때문에)
2. 버튼의 `인터랙터블 = false`

```cs
// 버튼
public void UpdateSpeedUpButton(bool isSpeedUp)
{
	// 상호작용 여부
	currentSpeedButton.interactable = StageManager.Instance.SlowState ? false : true;
	
	// 현재 배속 상태 반영
	currentSpeedText.text = isSpeedUp ? "x 1.5" : "x 1";
	currentSpeedImage.sprite = isSpeedUp ? x2SpeedSprite : x1SpeedSprite;
}
```

```cs
public void ToggleSpeedUp()
{
	// 현재 느려진 상태라면 별도의 처리를 하지 않음
	if (_slowState) return;

	if (_currentGameState == GameState.Battle || _currentGameState == GameState.Paused)
	{
		IsSpeedUp = !IsSpeedUp;
	}
}
```
#### 이후
- 클릭된 `DeployableBox`가 완전히 올라온 다음에 배속 상호작용 여부가 꺼진다. 클릭되자마자 `_slowState`를 활성화하는 식으로 바꿈
	
```cs
public void Select()
{
	IsSelected = true;
	AnimateSelection();
}

private void AnimateSelection()
{
	// 애니메이션을 위한 기존 위치 저장
	originalPosition = GetComponent<RectTransform>().anchoredPosition;
	isOriginalPositionSet = true;

	currentTween?.Kill();

	Sequence sequence = DOTween.Sequence();

	RectTransform rectTransform = GetComponent<RectTransform>();

	sequence.Append(rectTransform.DOAnchorPosY(originalPosition.y + animationHeight, animationDuration / 2)
		.SetEase(Ease.OutQuad));

	currentTween = sequence;
}
```
> 시퀀스가 끝나자마자 추가로 동작하는 로직이 있을 것 같았는데, 의외로 없다.

- `DeploymentInputHandler`에서 `timeScale`을 설정하는 코드 제거

- 박스를 클릭할 때마다 버튼이 활성화 / 비활성화를 반복함. 한 박스를 클릭한 상태에서 다른 박스를 클릭했을 때 해당 동작은 없애고 싶음
#### 정확한 원인을 모르겠어서 AI에게 던져줘봄
- 스크립트 3개짜리인데다가 애니메이션이 끝나고 추가로 동작하는 개념도 아니라서 잘 모르겠다.
- 정작 저 스크립트 3개에 있지도 않았다.  ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ
	- `DeployableBox, DeploymentInputHandler, DeployableManager`

#### CameraManager.AdjustForDeployableInfo
```cs
public void AdjustForDeployableInfo(bool show, DeployableUnitEntity? deployedDeployable = null)
{
	// ...
	
	if (show)
	{
		// ...
		// 카메라 이동 및 회전 (+ 후에 Slow 설정)
		_currentCoroutine = StartCoroutine(LerpPositionAndRotation(MainCamera.transform, newPosition, infoRotation, animationDuration, 
			onComplete: () => StageManager.Instance.SlowState = true));
	}
	// 해제 상황 : HideDeployableInfo
	else
	{
		// 카메라 이동 및 회전 (시작 시에 Slow 해제)
		_currentCoroutine = StartCoroutine(LerpPositionAndRotation(MainCamera.transform, originalPosition, baseRotation, animationDuration,
			onStart: () => StageManager.Instance.SlowState = false ));
	}
}
```
> 여기서 `onComplete`에 있는 부분 때문에 그렇다. 

근데 저걸 `onStart`로 바꾸면 카메라가 이동하는 과정 자체가 엄청 느려질 거 같은데?

- 해결 : `LerpPositionAndRotation`을 보면
```cs
    private IEnumerator LerpPositionAndRotation(Transform transform, Vector3 targetPosition, Quaternion targetRotation, float duration, Action onStart = null, Action onComplete = null)
    {
        float time = 0;
        while (time < duration)
        {
            time += Time.deltaTime // <----------------------
            yield return null;
        }
    }
```
> - **`time += Time.deltaTime` 부분을 `unscaledDeltaTime`으로 고치면 된다.**
> - 그리고 **`show` 부분도 `onStart`으로 바꿔주면 됨.**
> 	- 버튼이 깜빡이는 현상은 카메라가 도는 시간 동안 `SlowState = false`이기 때문인데, 코드로 처리하는 아주 짧은 시간이면 다시 `SlowState = true`가 되면서 버튼이 깜빡이지 않게 됨

- 더 나아가면, 저 코드는 **`CameraManager`에 있기 때문에 저기서 게임의 속도를 결정하는 것 자체가 별로다.** 차라리 저 메서드를 호출하는 쪽에서 `SlowState`를 설정하는 게 더 좋을 거임
	- 이 경우도 복잡하다. `StageUIManager`랑 `InStageInfoPanel` 등에서 다 호출하고 있기 때문. 이 부분도 정리를 좀 해봄.

- `CameraManager.AdjustForDeployableInfo`를 사용하는 스크립트들
	- `StageUIManager` : `ShowUndeployedInfo, ShowDeployedInfo`
	- `InstageInfoPanel` : `UpdateUndeployedInfo, UpdateDeployedInfo, Hide`
	- **매니저에서 패널의 메서드를 호출하는 형태이므로, 패널에서 카메라 매니저에 접근할 필요는 없다.** 
	- `SlowState`를 변경하는 곳은 `StageUIManager`에서만 호출하면 충분할 듯.
