```python
df.groupby(['col1', 'col2']).cumcount()
```

```python
# 이거 자체는 df.groupby().cumcount() + 1 을 적용한 결과임

				col1    col2    counter
4/20/2012        A      BL        1
4/27/2012        A      BL        2
5/4/2012         A      BL        3
5/11/2012        A      BL        4
5/18/2012        A      BL        5
4/20/2012        A      lk        1
4/27/2012        A      lk        2
5/4/2012         A      lk        3
5/11/2012        A      lk        4
5/18/2012        A      lk        5
5/25/2012        A      lk        6
```
- `2022 derby` 하다가 막혔던 부분임 : 어떻게 하면 각 그룹별로 Auto Increment를 적용할 수 있을까?
- 이걸 몰라서 겁나 헤맸다. 구글링은 항상 꼼꼼하게 합시다.
- `cumsum()`도 이렇게 하면 되겠죠?