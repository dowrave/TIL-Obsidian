```ts
const WritePost = (test: boolean) => {
	...
}
```
- 위와 같은 컴포넌트가 있다고 합시다
- 예를 들어 `App.tsx` 등에서 test의 값을 넣고 싶다고 가정해봅시다


```tsx
<Route path='test' element={<WritePost test={true}/>} />
<Route path=":section/post/create" element={<WritePost test={false}/>} />
```
- 이런 식으로.

그런데 위처럼 `test: boolean`을 넣으면 정상적으로 작동하지 않는데, 아래처럼 수정하면 됩니다.
```tsx
type WritePostProps = {
    test: boolean;
}

const WritePost = ({test}: WritePostProps) => {
	...
}
```

