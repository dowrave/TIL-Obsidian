- `useEffect`와 `useState`을 사용해 어떤 변수에 초기값을 할당하려고 하는데, 배열을 정상적으로 받아옴에도 변수에 초기값을 할당하지 않는 문제가 있다.
```tsx
    const fetchSubsections = async () => {
        try {
        const response = await axios.get<SubSection[]>(backend + `api/blog/subsection?section=${section}`)
        setSubsections(response.data.subsection)
        if (subsections.length > 0) {
            setSubsection(subsections[0].name)
        }
        } catch (e) {
            alert(`현재 ${e} 에러 발생 중`)
        }
    }

	
    useEffect(() => {
        fetchSubsections();
        // 가져온 1번째 subsection을 기본 subsection으로 지정함
    }, [])
```
---
- [[4. 리액트 컴포넌트#이벤트 핸들러와 상태 변경]]을 참고하면 좋다 
- **★★세터 함수는 비동기로 작동한다.** 즉, 상태의 변경은 그때그때 일어나지 않는다.
- 그럼에도 그러한 기능이 필요하다면, `useEffect(~~~, [subsections])`처럼 초기화되기를 원하는 배열이 초기화된 다음 함수를 실행하도록 구현할 수 있다.

```tsx
    const fetchSubsections = async () => {
        try {
          const response = await axios.get(backend + `api/blog/subsection?section=${section}`);
          setSubsections(response.data.subsection);
        } catch (e) {
          alert(`현재 ${e} 에러 발생 중`);
        }
      }

	useEffect(() => {
        fetchSubsections();
    }, [])

// subsections이 들어오면 실행됨
    useEffect(() => {
        if (subsections.length > 0) {
            setSubsection(subsections[0].name);
        }
    }, [subsections])
```