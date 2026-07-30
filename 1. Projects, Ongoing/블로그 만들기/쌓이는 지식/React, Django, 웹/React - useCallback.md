- 함수를 캐시하며, 의존성 배열이 바뀌었을 때만 재계산된다.
	- 바뀌지 않고 같은 값이 들어온다면 연산 없이 결과만 반환하여 자원을 아낌
```jsx
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```
> `a, b`가 바뀔 때만 재계산되는 함수이다.