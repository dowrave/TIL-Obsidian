```python
my_order = ['Low', 'Medium', 'High']
pd.factorize(my_order)
## (array([0, 1, 2], dtype=int64), array(['Low', 'Medium', 'High'], dtype=object))
```
- 2개의 `array`를 반환함. 