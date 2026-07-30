> 요약
> 1. UI 레이캐스트는 3D 레이캐스트보다 먼저 실행된다.
> 2. 주로 Canvas 위의 요소들에 대해 레이캐스트가 실행되는데, 이 때 Canvas는 `Graphic Raycaster`라는 컴포넌트가 활성화되어 있어야 한다.



## UI 시스템
- Unity의 UI 시스템은 기본적으로 `GraphicRaycaster`와 `EventSystem`을 사용합니다.
- **UI 요소에 대한 레이캐스트는 일반적으로 3D 오브젝트에 대한 레이캐스트보다 먼저 수행**됩니다.
- 'Screen Space - Overlay'인 경우, UI는 항상 3D 오브젝트 위에 그려지며, 레이어 설정과 관계없이 먼저 검사됩니다.

### UI 요소
- 일반적으로 `Canvas` 게임 오브젝트 아래의 오브젝트들을 의미하지만, 모든 게 대상은 아니다.
- 주요 요소들 (**Raycast Target 속성이 활성화**되어 있다)
	- `Graphic` 컴포넌트를 가진 오브젝트(`Image, Text, RawImage`)
	- `Selectable` 컴포넌트를 상속받은 오브젝트(`Button, Toggle, Slider, Dropdown`)
	- `CanvasRenderer` 컴포넌트를 가진 오브젝트들

## 레이캐스트 동작 방식
- `EventSystem`이 먼저 UI 요소에 대한 레이캐스트를 수행합니다. 
	-  UI 요소가 클릭되면, 이벤트가 처리되고 일반적으로 3D 오브젝트에 대한 레이캐스트는 수행되지 않습니다.
- UI 요소가 없는 영역을 클릭하면, EventSystem은 UI 히트를 감지하지 못하고, 그 다음 3D 오브젝트에 대한 레이캐스트가 수행됩니다.

## Physics.Raycast vs EventSystem
- `Physics.Raycast`는 3D 공간의 콜라이더에 대해 작동하며, UI 요소를 감지하지 않습니다.
- UI 요소는 `EventSystem.current.IsPointerOverGameObject()`나 `GraphicRaycaster`를 통해 감지해야 합니다.

## Canvas 설정
- `Canvas`의 설정에 따라서도 반응이 달라질 수 있다.
	- `Overlay`: 이 모드에서는 UI 요소가 항상 3D 오브젝트 위에 렌더링되며, 레이캐스트도 우선적으로 처리됩니다.
	- `Camera`: 이 모드에서도 UI가 3D 공간 위에 렌더링되지만, 특정 카메라에 연결됩니다.
	- `World Space`: 이 모드에서는 UI가 3D 공간 내에 존재하며, 다른 3D 오브젝트와 동일하게 취급될 수 있습니다.

UI 요소의 레이캐스트 우선순위는 주로 **Canvas의 'Sort Order'와 개별 UI 요소의 계층 구조에 따라 결정**됩니다. 높은 Sort Order를 가진 Canvas의 요소가 먼저 레이캐스트되며, 같은 Canvas 내에서는 계층 구조상 더 위에 있는(나중에 그려지는) 요소가 우선됩니다.