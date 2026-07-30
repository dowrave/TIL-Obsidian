
- - - 생각보다 유용하진 않아서 밑으로 빼뒀다.
```python
a = df.groupby('col')

a.groups.keys() # 이건 말 그대로 어떤 그룹으로 묶었는지 보는 거임
a.groups.values()
```
-  `groupby`가 `key, value`로 묶인다

### 활용하기
```python
a = df.groupby('col')
keys = a.groups.keys()

for i in keys:
	print(a.get_group(i))
```
`groupby` 오브젝트에 `get_group(key)`을 적용하면 해당 key로 묶인 그룹만을 볼 수 있음
