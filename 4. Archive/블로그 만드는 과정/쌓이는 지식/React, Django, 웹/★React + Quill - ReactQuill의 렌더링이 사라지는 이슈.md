- 며칠 동안 골머리를 앓다가 일단 방법은 알아냄

## 문제 상황
- `PostDetail -> QuillRead -> ReactQuill` 순서대로 부모 -> 자식 컴포넌트로 구성된 상황이다.
- 글의 헤더 부분을 따로 추려내서, `PostDetail`의 오른쪽 부분에 헤더 위치로 스크롤되는 링크를 생성하려고 했으나, **글이 `ReactQuill`에 들어간 상황에서 부모 컴포넌트인 `PostDetail`을 업데이트하면 `ReactQuill` 컴포넌트가 사라지는 이슈가 발생**했다.

## 원인
- 정확한 문제는
1. `PostDetail`에서 글을 받은 뒤, `QuillRead`로 `content`를 전달한다. 이 때, 하위 컴포넌트의 로딩이 완료되었음을 알리는 변수와 함수도 같이 전달한다. 이는 `useState`를 쓰든 `Redux`를 쓰든 비슷비슷함.
2. 하위 컴포넌트인 `QuillRead`에서 `useEffect` 훅 내부에서 전달받은 핸들러 함수를 이용한다.
3. **핸들러 함수를 통해 상태를 변경하는데, 컴포넌트의 상태 변경은 리렌더링을 유발한다.** 이 과정에서 기존에 렌더링되었던 `ReactQuill, QuillRead`가 사라지는 현상이 발생한다.
## 해결
- **상태 변경이 리렌더링을 유발하는 건 리액트의 기본 동작**이다.
- 상태를 바꾸더라도 렌더링을 다시 유발시키지 않기 위해, `React.memo`와 `useCalllback` 모두를 쓸 필요가 있다.

```tsx
// PostDetail.tsx
    const [isQuillLoaded, setQuillLoaded] = useState(false);

    const handleQuillLoaded = useCallback(() => {
        if (!isQuillLoaded){
            // dispatch(setQuillLoaded(true));   
            setQuillLoaded(true);   
            }  
        
        }, [])
    
	...
	
	useEffect(() => {
        console.log(isQuillLoaded)
    }, [isQuillLoaded])

	...
	return (
	...
	<QuillRead htmlContent = {post?.content}
                               isLoaded = {handleQuillLoaded}
                    />
	)

// QuillRead.tsx
const QuillRead = React.memo(({ htmlContent, isLoaded }) => {
	...
	useEffect(() => {
        isLoaded()
    }, [])
    
	return (
    <div>
        <ReactQuill 
        modules={modules} 
        value={htmlContent} 
        readOnly={true} 
        theme='snow'
        />
    </div>
    )

```
> - 내 프로젝트는 `Redux`를 이용해 상태와 함수를 관리했지만, `useState`를 써도 잘 작동했다. 차이점이라면 `setQuillLoaded`에 `dispatch`를 붙이냐(Redux) 아니냐(useState) 차이.
> - 상위 컴포넌트의 `useCallback`, 하위 컴포넌트의 `React.memo` 모두가 존재해야 잘 작동했다. 하나라도 없으면 리렌더링되면서 컨텐츠 내용이 날아갔음.

### React.memo
- **컴포넌트의 `props`가 변경되지 않으면 해당 컴포넌트의 리렌더링을 방지**한다.
- `얕은 비교`를 수행하며, `props`가 객체나 배열이라면 그 내부 구조의 변경은 감지하지 못한다.

### useCallback
- 함수를 `메모이제이션Memoization`하는데 사용된다. 
- **일반적으로 컴포넌트가 리렌더링되면 내부에 정의된 함수도 새롭게 생성**되지만, `useCallback`은 `useEffect`처럼 의존성 배열을 인자로 받으며 **의존성 배열 내부의 원소가 변경될 때만 함수가 재생성**된다. 