```tsx
{data.map(({ day, count }) => (
	<div>
		<div
		key={day}
		className='activity-box relative z-40'
		style={{
			color: bgColorMax,
			filter: `saturate(${calculateSaturation(count, maxCount)}%)`
		}}>
			<div className='activity-box-message absolute z-50'>
				{`${getDateFromCount(year, count)}일 작성된 글은 ${count}개입니다.`}
			</div>
		</div>
	</div>
))}

```
> 자식 요소의 `div`가 더 높음에도 반복문으로 구현된 다른 부모 요소를 가리는 이슈가 있었음
> 한편, 직접적인 부모 요소에 대해서는 z-index가 정상적으로 적용되었음

## 해결법
- `스택 컨텍스트` 관련 이슈인데, 왜 이 방법이 먹히는지 정확하게 이해하진 못했다. 
	- 별도의 스택 컨텍스트에서는 `z-index`가 먹히지 않는다는 것인데, `relative`에서 스택 컨텍스트가 생성된다고 한다. 즉 여전히 **별도의 스택 컨텍스트 간에서의 렌더링 우선 순위를 비교한다는 점에서는 변함이 없어 보이기 때문**이다. 
	- "**자식 컴포넌트였던 `message`를 형제 컴포넌트로 올리고, 그 위에 `div relative`를 추가하는 방식이 통했다**" 정도만 정리하고 넘어가겠다.

- 따라서, 아래처럼 형제 관계로 수정하고, `index.css`도 바꿔서 해결했다.
```tsx
<div className='relative'>
	<div
	key={day}
	className='activity-box z-40 '
	style={{
		color: bgColorMax,
		filter: `saturate(${calculateSaturation(count, maxCount)}%)`
	}}>
	</div>
	<div className='activity-box-message z-50'>
		{`${getDateFromCount(year, count)}일 작성된 글은 ${count}개입니다.`}
	</div>
</div>
```
```css
.activity-box {
  width: 10px;
  height: 10px;
  @apply relative bg-gray-300 rounded-sm;
}

.activity-box-message {
  @apply hidden absolute top-full left-0 bg-gray-500 p-2;
}
/* 형제 요소로 변경*/
.activity-box:hover + .activity-box-message {
  @apply block
}
```
> 스택 컨텍스트는 `relative`를 가진 `div`에서 생성된다. 

### 스택 컨텍스트
- 웹 페이지의 특정 영역에 대한 렌더링 순서를 제어하는 개념이다.
- 즉, 어떤 요소가 다른 요소 위에 표시될 때 무엇을 우선시할지에 대한 개념이다.

- 각 스택 컨텍스트는 독립적으로 생성되며, 각 스택 컨텍스트 내부에서는 `z-index` 등을 활용해 렌더링 순서를 제어할 수 있다. 
- 그러나, **한 스택 컨텍스트의 내부는 다른 스택 컨텍스트와는 별개로 렌더링된다.**  이 경우, 소스 코드에 먼저 선언된 요소가 먼저 렌더링된다고 한다. 따라서, 전체적으로 봤을 때 `z-index`가 더 높은 요소이더라도, 스택 컨텍스트가 다르다면 렌더링 우선순위에서 밀리는 경우도 있다.
	- (?) : 그런데 위 설명에 의하면 **바꿔서 구현된 요소도 결국 다른 부모를 가진 자식 간의 렌더링 우선순위 비교니까 z-index로 비교하는 게 아닌데, 어떻게 message가 box보다 우선 순위에 나오는거임**?