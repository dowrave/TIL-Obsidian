`stringArray = ["1", "2", "3", "4", "5"];`

위 배열의 원소들을 숫자로 변환하기
## 파이썬
```python
numberArray = list(map(int, stringArray))
print(numberArray)
```

## 타입스크립트
```tsx
const numberArray = stringArray.map((value) => parseInt(value));
```