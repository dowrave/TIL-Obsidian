```tsx
// 함수를 onClick에서 바로 호출
<button onClick={handleWritePost(subject, category)}></button>

// 함수 레퍼런스만 전달
<button onClick={() => handleWritePost(subject, category)}></button>
```

- 위 2개에는 큰 차이가 있으며, 일반적으로 아래처럼 구현하는 게 올바른 구현이다.
1. 함수를 바로 호출 : **페이지를 로드하는 과정에서 함수가 바로 실행**된다.
2. 함수 레퍼런스만 전달 : **해당 요소를 클릭할 때만 함수가 실행**된다.