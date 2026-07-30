- 프로젝트 중, `signin` 후 `useNavigate`될 위치를 현재 활성화된 내비게이션바`category`의 홈 위치로 지정하고 싶었다.
- 이를 위해선 `navbar.tsx`에서 `useState`로 관리되는 `category`를 전역 상태로 뺄 필요가 있었고, (어차피 할 예정이었던) `Redux`로 통합하기로 했다.


---

## 기존 코드
- `navbar.tsx`
```tsx
  const [category, setCategory] = useState<string>(useParams().category || 'work');

// ...
// 토글 버튼 클릭으로 카테고리 변화
  const handleCategoryClick = (newCategory) => {
    setCategory(newCategory);
  }; 
```
- [[React - 상태 관리 - redux로 이사가기#3. 액션 처리 및 상태 변화]]와 비교해보자.

## 변경

### 1. 리덕스 스토어 설정

- `rootreducer.ts`
```tsx
import { combineReducers } from '@reduxjs/toolkit'
import authReducer from './reducers/authReducer'
import categoryReducer from './reducers/categoryReducer';

const rootReducer = combineReducers({
    auth: authReducer,
    category: categoryReducer,
});

export default rootReducer // store.ts에서 가져감
```

- `store.ts`
```tsx
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './rootReducer';

const store = configureStore({
    reducer: rootReducer,
});

export default store; // main.tsx에 store로 제공됨
```
> `store` : 앱의 상태를 보관하는 곳이다. 모든 상태 변경은 store를 통해 이뤄지며, 앱의 전체 상태트리를 유지한다. 
### 2. 액션과 리듀서 정의

- `categoryReducer.ts`
```tsx
// 액션 타입
const SET_CATEGORY = 'SET_CATEGORY';

// 액션 생성 함수. useDispatch로 이용된다.
export const setCategory = (category) => ({
    type: SET_CATEGORY,
    payload: category,
})

// 초기 상태
const initialState = {
    category: 'work'
}

const categoryReducer = (state = initialState, action) => {
    switch (action.type) {
        case SET_CATEGORY:
            return {
                ...state,
                category: action.payload,
            };
        default:
            return state;
    }
}

export default categoryReducer;
```
> 굳이 액션 타입을 정의하는 이유는, `switch` 문을 구현하는 편이 더 가독성이 좋고 유지보수가 간편하기 때문이다.
> 만약 상태 변경 로직이 다양하다면, 리듀서로 관리되는 다양한 상태 변경 로직 각각에 접근할 용도가 필요하고, 이를 식별하기 위한 **식별자**로 타입을 생각해도 무방할 것 같음. 
> 구조가 낯설어서 유독 서술이 길어졌다.

### 3. 액션 처리 및 상태 변화
- 상태는 `useSelector`을, 상태 변화는 `reducer`을 통해 진행한다.
- [[React - 상태 관리 - redux로 이사가기#기존 코드]]와 비교해보자.

```tsx
// useState 대신 useSelector 및 useDispatch를 쓴다
import { useSelector, useDispatch } from 'react-redux';
import { setCategory } from './path/to/categoryReducer'; // 리듀서에서 정의한 액션

  const category = useSelector(state => state.category.category);
  const dispatch = useDispatch();

  const handleCategoryClick = (newCategory) => {
    dispatch(setCategory(newCategory));
  }; 
```

### useSelector 
- Redux 스토어의 상태에 접근하기 위해 사용한다.
- **스토어의 전체 상태`state`를 인자로 받아, 그 중에 필요한 부분만을 선택하여 반환**한다.
- **일반적으로 `state.리듀서.상태` 형식으로 접근한다.** 
	- 즉 위에선 `category` 리듀서(`rootreducer.ts`에 있는)의 `category` 상태에 접근하는 것이다. 

### useDispatch
- Redux 스토어에 대한 디스패치 함수에 접근할 수 있게 해준다. 액션을 디스패치하여, 상태를 변경하는 역할을 한다.
- `디스패치Dispatch`란, 액션을 스토어에 보내는 과정을 의미한다.
	1. 액션 생성 : 액션 생성자를 사용해 액션 객체를 만든다. 수행할 작업의 종류를 의미하는 `type` 필드를 포함한다.
	2. 생성된 액션 객체는 `dispatch` 함수를 통해 Redux 스토어로 전달된다.
	3. 스토어는 디스패치된 액션을 받아 설정된 리듀서 함수에 액션을 전달한다. 리듀서 함수는 액션 타입에 따라 앱의 상태를 새로운 상태로 계산하여 반환한다.
	4. 리듀서에 의한 반환된 새로운 상태는 스토어에 저장되며, 앱의 상태가 업데이트된다.



