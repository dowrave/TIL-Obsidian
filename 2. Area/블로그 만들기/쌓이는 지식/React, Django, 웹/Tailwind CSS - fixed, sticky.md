- `fixed`는 스크롤에 관계 없이 화면의 특정한 위치에 고정된다.
	- 이 때, 해당 요소는 **상위 요소의 영향을 받지 않는 상태**가 된다.

- `sticky`는 일정 위치까지 스크롤 되고 그 이후로는 없어지지 않는 상태가 된다.

- 즉 아래 상황에서 차이가 생길 수 있다.
```
flex
  flex-none
    sticky
  flex-1

flex
  flex-none
    fixed
  flex-1
```
> - `fixed`의 경우, `flex` 박스를 이탈하므로 2번째의 `flex-1`이 위에 있는 `flex-none`의 공간을 침범할 수 있다.
> - 따라서 `fixed`로 구현하고 싶다면 `flex-none`에 해당하는 영역의 높이를 구한 다음, `flex-1`에 `margin top(mt)`를 줘야 할 것으로 보임.

