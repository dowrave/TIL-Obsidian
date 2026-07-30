```python
spec = dict(x="method", y="distance", data=planets)
sns.stripplot(**spec, size=4, color=".7")
sns.pointplot(**spec, join=False, ci=0, capsize=.7, scale=0)
```
- 키워드를 반복해서 전달하는 게 번거로울 수 있음
- 이 때 사용할 수 있는 테크닉: `dict`로 `key-value`쌍을 만들어 변수로 저장하고, 해당 키워드 인자가 쓰이는 곳에 `**`을 붙여 전달하는 방식임.