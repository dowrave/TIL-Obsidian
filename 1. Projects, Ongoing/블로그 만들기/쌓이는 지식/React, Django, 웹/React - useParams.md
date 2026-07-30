- 리액트 라우터에서, 동적 라우트의 동적 세그먼트에 해당하는 부분을 추출한다.

- 예를 들어 `App.tsx`에서 
```tsx
<Route path='/work' element={<WorkLayout />}>
  <Route path='study' element={<WorkStudy />} />
  <Route path='study/:id' element={<PostDetail />} />
</Route>
```
> 이렇게 구성된 라우터가 있다고 하자. 

- 이 때, `PostDetail.tsx`의 `useParams`부분은
```tsx
const { id } = useParams<{ id: number }>(); 
```
> 이렇다고 하면, `useParams`은 `App.tsx`에 있는 `:id` 를 추적, 그 곳에 해당하는 값을 추출하게 된다. 타입스크립트의 경우에만 자료형 지정`id: number`이 가능하다.


## 이 기능은 중첩 라우트에도 적용이 가능하다!
- 쥰내 쎈 기능입니다.
```tsx
        <Route path=":subject/*" element={<Layout />}>
          <Route path=":category/:page" element={<PostList />} />
        </Route>
```
> 이렇게 구성할 경우, `PostList` 컴포넌트에서는

```tsx
const { subject, category, page } = useParams();
```
> 으로 상위 컴포넌트에 있는 `subject`까지 정보들을 싹 다 읽어올 수 있습니다.



