```python
# 외곽선 지우기
plt.gca().spines['tblr'].set_visible(False)
# tblr : top / bottom / left / right
```
- `plt.gca()` 는 `ax[]`로 치환 가능함



```python
# 눈금 지우기
plt.yticks(ticks = [])

# 근데 subplots에 그린 경우는 어떡함?
ax.get_xaxis().set_visible(False)
ax.get_yaxis().set_visible(False)

# 아니면 이렇게도 가능하다
ax.tick_params(bottom = False,
			  labelbottom = False)

```

