- 파이썬의 f-string처럼 쓰려다가 어떤 이슈가 발생했음
```tsx
// response는 axios.get으로 받은 게시판 정보(post, total_pages)
setPosts(response.data.posts);
```
> 여기서 위 정보를 **리터럴로 출력하면 `[object Object]`처럼 정체를 알 수 없게 나타난다.**
```tsx
console.log(`response : ${response.data.posts}, ${response.data.total_pages}`); // [object Object]

// ``을 쓰는 대신, ""을 써서 제대로 출력시키면 됨
console.log('response posts : ', response.data.posts);
```


