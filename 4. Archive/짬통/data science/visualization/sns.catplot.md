- 참고하고 싶은 열이 3개 이상일 때 사용할 수 있음

```python
sns.catplot(data = ,
		   x = ,
		   y = ,
		   hue = ,
		   row = ,
		   col = ,
		   kind = , # 그래프 종류
		   order = , # x축 label 순서
		   )
```

- `hue_order` 또한 존재함
-  `FacetGrid`와의 차이점은 아직 몰?루 - 근데 FacetGrid로 못 그렸던 걸 여기선 그릴 수 있었다.
- 예제를 보면 여러 플롯을 동시에 그리는 데에도 쓰이는 듯 하다