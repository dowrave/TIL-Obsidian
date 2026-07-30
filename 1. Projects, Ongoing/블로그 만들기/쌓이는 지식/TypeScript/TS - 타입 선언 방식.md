- 여기선 크게 `<> : 제네릭 선언` 및 `: - 타입 지정` 2가지를 예제와 함께 살펴봄


- 예시
```tsx
function identity<T>(arg: T): T {
    return arg;
}

function identity(arg: T): T{
	return arg;
}
```
- `<>`에 대해서는 [[TS - 제네릭]] 참고
## 1. 제네릭 선언 `<>`
```tsx
function identity<T>(arg: T): T {
    return arg;
}
```
- `<T>`는 제네릭 타입의 매개 변수를 정의하는 부분이다 
- `identity` 함수가 호출될 때, 어떤 타입이든 받을 수 있으며, 해당 타입은 함수 내에서 `T`로 참조된다. 반환 타입 또한 `T`로 지정된다.

## 2. 타입 지정 `:`
```tsx
function identity(arg: T): T{
	return arg;
}
```
- `T` 타입에 대한 정의가 명시되지 않았다면, 타입스크립트 컴파일러는 `T`를 인식하지 못하여 오류를 발생시키게 된다. 