- 예전에 했던 타이타닉 예제에서 가져옴
```python
grid = sns.FacetGrid(train_df, 
					 col='Survived', 
					 row='Pclass', 
					 height=2.2, 
					 aspect = 1.6)
grid.map(plt.hist, 
		 'Age', 
		 alpha = .5, 
		 bins=20)
grid.add_legend()
```
- `sns.FacetGrid` 객체를 만들고, 그 속에 어떤 플롯을 넣을지 결정할 수 있는 듯