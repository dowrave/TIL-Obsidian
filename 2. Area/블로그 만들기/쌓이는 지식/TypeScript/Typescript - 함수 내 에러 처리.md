```ts
const func1 = () => {
	try {
		func2();
		console.log("Hello World!")
	} catch (e) {
		// blabla
	}
}

const func2 = () => {
	try{
		if (조건) {
			throw Error("엘렐레")
		}
	} catch (e) {
		// blabla2
	}
}
```
> 함수 1이 2를 포함하고 있으며, 두 함수 모두 try - catch문을 갖고 있다.

- 만약 함수 2에서 에러가 발생한 경우, 어떻게 처리될까?


- 함수 2에서는 에러를 발생해서 catch 문을 타지만, 함수 1에서는 func2()가 수행된 다음, `console.log()`문이 실행된다.
- 즉, 내부 함수에 `try - catch` 문이 있고 **내부 함수에서 오류가 발생했다면, 외부 함수에서는 내부 함수의 오류를 인식하지 못하고** 정상적으로 작동했다고 인식, **그 다음 코드를 실행하게 된다.**

- 만약 `func2`의 에러를 `func1`에서 캐치하게 하고 싶다면, `func2`의 `try-catch`문을 제거하면 된다.
```ts
const func2 = () => {
	if (조건) {
		throw Error("엘렐레")
	}
}
```
