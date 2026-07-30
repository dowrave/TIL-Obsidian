1. Python
	- 구분자를 지정하지 않으면 기본적으로 공백 문자(스페이스, 탭, 개행 문자)를 기준으로 함
	- 최대 분리 횟수는 `maxsplit`으로 지정

2. TypeScript
- **구분자를 반드시 지정해야 함**
```tsx
text.split(",")
text.split("\n") 
```
- 첫 문자, 마지막 문자가 공백이라면 `split` 함수의 결과 배열의 원소에 `''`이 포함됨


### 첫 문자 / 끝 문자가 공백인 경우(백준 1152번)
" The first Character is a blank"라는 문장에 대해

- 파이썬
```python
['The', 'First', 'Character', 'is', 'a', 'blank']
```

- 타입스크립트
```ts
[ '', 'The', 'Curious', 'Case', 'of', 'Benjamin', 'Button' ]
```
> 이런 값들을 제거하려면 `string.trim()`후 `split(" ")`을 해주면 된다.
> **`string.trim()` 함수는 문자열 양쪽 끝의 공백 문자를 제거한다.**


