- 백엔드에서 데이터를 불러온 뒤, 이들을 프론트에서 보여주는 과정이 아래와 같음
```tsx
    useEffect(() => {
        fetchData();
    }, [id]);

    const fetchData = async () => {
        try { 
            console.log(`http://localhost:8000/api/blog/post/?id=${id}`)
            const response = await axios.get<Post>(`http://localhost:8000/api/blog/post/?id=${id}`)
            console.log(response)
            setPost(response.data)
        } catch (e) {
            alert(`현재 ${e} 에러 발생 중`)
        }
    };
```

- 그런데 프론트에서 데이터를 보여줄 때, HTML에 직접 작성한 요소가 먼저 나타나고 **백엔드에서 불러온 데이터는 나중에 렌더링되는 현상**이 있었음
- 렌더링되는 순서는 아래와 같음

> 1. **최초 렌더링:** 컴포넌트가 최초로 렌더링될 때, `useEffect`의 콜백 함수가 실행됩니다.
> 2. **`fetchData` 호출:** `useEffect` 콜백 함수 내에서 `fetchData` 함수가 호출되고, 비동기적으로 데이터를 가져오기 시작합니다.
> 3. **렌더링:** `fetchData` 함수는 비동기이기 때문에 렌더링이 끝나도 데이터가 아직 완전히 로드되지 않았을 수 있습니다. 그래서 `{}` 안의 내용은 처음에는 데이터가 없는 상태로 렌더링됩니다.
> 4. **데이터 로드 완료:** `fetchData` 함수가 완료되고 데이터가 로드되면 `setPost(response.data)`가 호출되어 상태가 업데이트됩니다.
> 5. **재랜더링:** 상태가 업데이트되면 컴포넌트가 다시 렌더링되고, 이번에는 `{}` 안의 내용이 데이터가 있는 상태로 렌더링됩니다.

---
### 해결법
- `useEffect(() => ({function}, [])`처럼, 의존성 배열을 없애는 방법이 있다.
	- 의존성 배열을 없애면 `useEffect` 훅은 마운트될 때와 언마운트 될 때 2번만 작동하게 된다.
```tsx
    useEffect(() => {
        fetchData();
    }, []); // 의존성 배열 제거

    const fetchData = async () => {
        try { 
            console.log(`http://localhost:8000/api/blog/?category=${category}`)
            const response = await axios.get<Post[]>(`http://localhost:8000/api/blog?category=${category}`)
            setPosts(response.data)
        } catch (e) {
            alert(`현재 ${e} 에러 발생 중`)
        }
    };
    
	// posts가 없다면 아무것도 렌더링되지 않는다
    if (!posts) {
        return null;
    }

```
> 이 방법은 PostDetail이나 PostList 모두 적용이 됐지만, **PostList -> PostList 페이지로 이동할 때 다른 카테고리의 페이지가 여전히 렌더링**되는 문제가 있다.
> - 페이지 이동 과정에서 컴포넌트의 상태가 유지되는 문제로, 컴포넌트가 언마운트될 때 이전 데이터를 초기화하는 작업이 추가로 필요하다.


- 결국 의존성 배열을 유지하되, `Loading` 화면을 별도로 추가해서 데이터를 모두 불러온 경우에만 화면을 보여주고, 이외에는 Loading 화면을 보여주는 방식으로 구현함.
```tsx
const PostList: React.FC<PostListProps> = ({ category }) => {
    const [posts, setPosts] = useState<Post[]>([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetchData();
    
        return () => {
            // 컴포넌트 언마운트 시 이전 데이터 초기화
            // 카테고리가 달라지는데도 이전 카테고리 게시판이 남는 현상 수정
            setPosts([])
        }
    }, [category]);

    const fetchData = async () => {
        try { 
            console.log(`http://localhost:8000/api/blog/?category=${category}`)
            const response = await axios.get<Post[]>(`http://localhost:8000/api/blog?category=${category}`)
            setPosts(response.data)
            setLoading(false);
        } catch (e) {
            alert(`현재 ${e} 에러 발생 중`)
        }
    };

    if (!posts) {
        return null;
    }

  return (
    <div>
        { loading ? (
            <p> Loading... </p>
        ) :
        (
        <>
            <h1>{category} 게시판</h1>
            <ul>
                {posts.map((post: Post) => (
                    <li key={post.id}>
                        <Link to ={`post/${post.id}`}>
                            {post.title}
                        </Link>
                    </li>
                ))}
            </ul>
        </>
        )
            }
    </div>
  )
}
```
> `useEffect` 문에 `return` 문을 남긴다면, 이 부분은 `정리Clean-Up` 함수를 정의하는 부분이다. 컴포넌트가 언마운트되거나 의존성 배열의 값이 변경되기 전에 실행된다.
> - 즉 이전 데이터를 초기화하고 싶다면 넣는 부분이다.

