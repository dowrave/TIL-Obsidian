- 배열에만 적용할 수 있음
- 배열 내에 있는 반복되는 요소가 객체일 때, **객체의 `key` 값을 명확히 지정해줘야 함**

- 예시
```tsx

data = [{day : 1, count : 0}, {day: 2, count: 1}, ...]

// 이건 정상 작동 : day와 count라는 key값을 명확히 지정해줬기 때문
data.map(({ day, count }) => {
	const [month, xx] = getDateFromCount(year, day)
})

// 한편 아래는 정상 작동하지 않음 : key 값을 별칭(date)으로 사용했기 때문에 undefined 출력
data.map(({ date, count }) => {
	const [month, xx] = getDateFromCount(year, date)
})
```
