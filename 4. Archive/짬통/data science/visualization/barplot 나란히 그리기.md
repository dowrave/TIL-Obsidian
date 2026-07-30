[원본링크](https://mirandaherr.tistory.com/35)
```python
x = np.array(range(len(df.columns)))
w = 0.3

# plot 1
plt.bar(x, 
		data.loc[:, 'col1'], 
		width = w, 
		label = 'label')
```
- `plt.bar()`에서 `x, data.loc[]`는 원래 들어가는 x, y값임

```python
x += w

# plot 2
plt.bar(x,
	   data.loc[:, 'col2'],
	   width = w,
	   label = 'label')
```