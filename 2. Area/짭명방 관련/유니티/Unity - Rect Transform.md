#Unity #유니티UI 

## Rect Transform

![[Pasted image 20260922153956.png|577]]

- `Anchor, Pivot`이 볼 때마다 헷갈리므로 특히 그 두 부분에 대해 정리함
- 둘 모두 **기준 좌표는 (0, 0) - 좌측 하단**
### Anchor(Min, Max)
- **부모 안에서 이 UI가 어디에 붙어 있는가?** 에 관한 기준. 
- 부모 기준의 정규화 좌표. (0~1)
- 정확하게는, **부모 안에 `앵커 영역Anchor Rectangle`이라는 가상의 사각형을 정의하는 값**이다. 두 값이 완전히 같은지, 다른지 여부에 따라 앵커 영역을 쓰는 방식이 달라진다.

![[anchor_min_max_two_cases.svg|577]]
#### AnchorMin == AnchorMax 동작(점 앵커)
- 앵커 영역이 넓이 = 0인 점 하나로 쪼그라든다.
- 해당 축에 대해, 부모 영역과 무관하게, RectTransform이 **고정된 크기**(width, height 등. 즉 sizeDelta)를 그대로 유지한다. 
- `AnchorMinX == AnchorMaxX`라면 상부에 `width`라는 값이 생긴다.`Y`라면 `height`가 생기는 개념.
- 부모가 리사이즈되면 앵커 점의 위치가 이동하지만, 크기는 그대로 유지된다.

#### AnchorMin != AnchorMax 동작(영역 앵커, 스트레치)
- `anchorMax ~ anchorMin` 사이에 실제 넓이를 갖는 영역이 생긴다.
- `RectTransform`은 **그 영역의 경계에 달라붙는다.**
- `Inspector`에서는 `Left/Right/Top/Bottom` (내부적으로는 `offsetMin, offsetMax`)이 생기며, 해당 경계에서 몇 픽셀만큼 뗄지를 의미한다.
- 부모의 리사이즈 시, 앵커 영역 자체가 비율대로 커지거나 작아지며, `RectTransform`도 그 오프셋을 유지한 채 같이 늘어난다.

### Pivot
- 이 RectTransform **자신의 기준점(원점)** 이 어디인가?
- 앵커가 잡힌 다음에 결정되는 값이다. 
- 자신 기준의 정규화 좌표. (0~1)
- 위치, 회전, 스케일이 적용되는 중심점이다. 

---

사실 이외에도 코드 단위로 만지면 `anchoredPosition`이나 `sizeDelta` 같은 부분들이 있긴 한데, 이건 나중에 천천히 다뤄보자...