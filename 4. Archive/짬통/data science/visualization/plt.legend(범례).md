- `ax` 객체에서 범례 없애기
```python
ax[i].legend().set_visible(False)
```

- `ax` 밖에 범례 표시하기
```python
box = ax.get_position()
ax.set_position([box.x0, box.y0, box.width * 0.8, box.height])
ax.legend(loc = 'center left', bbox_to_anchor = (1, 0.5))
```
[자세한 내용](https://stackoverflow.com/questions/4700614/how-to-put-the-legend-outside-the-plot)
