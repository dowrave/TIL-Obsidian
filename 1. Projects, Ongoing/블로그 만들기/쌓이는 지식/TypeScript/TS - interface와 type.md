- 타입스크립트에서 객체의 구조를 선언할 때, 
	- 처음에 공부하던 책에서는 `type`을 이용해 선언했고
	- `ChatGPT`에 물어보면 주로 `interface`로 답변을 주는 것을 볼 수 있었다.


- 심플한 결론
	- `interface` - 객체 구조 정의 & 확장 시 사용
	- `type` - 유니온, 튜플, 복잡한 타입 조작 등에서 사용

---
## 공통점
- 둘 모두 **객체의 형태를 정의할 때 쓸 수 있음**

## 차이점

### 1. 재선언(선언적 확장)
- 같은 이름의 객체를 **재선언(및 확장)하는 것은 Interface에선 가능하지만, Type에선 불가능**하다.
	- 이를 `선언적 확장Declaration Merging`이라고 한다.
	- 추가) `interface`가 그런 면에서 쓰기 편하지만, `typescript`를 굳이 쓰는 이유를 생각하면 **명시적으로 확장한다는 표현을 작성**하는 게 더 좋아보이긴 함 
### 2. 확장성(상속성)
- 둘 모두 가능하며, 방법에 차이가 있다.

- `interface`
```tsx

interface User {
	name: string;
}

interface UserDetail extends User {
	age: number;
}
```
> `extends`라는 키워드로 확장(상속)이 가능하다.

- `type`
```tsx
// type
type User {
	name: string;
}

type UserDetail = User & {
	age: number;
}
```
> 확장, 병합이 자동으로 이뤄지지 않으며, `&`을 이용해 타입을 확장할 수 있다.

