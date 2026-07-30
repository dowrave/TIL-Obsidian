```python
# plot swarmplot 
sns.swarmplot(data=tips, x="day", y="total_bill") # plot boxplot 
sns.boxplot(data=tips, x="day", y="total_bill", 
		showcaps=False, # 박스 상단 가로라인 보이지 않기 
		whiskerprops={'linewidth':0}, # 박스 상단 세로 라인 보이지 않기 
		showfliers=False, # 박스 범위 벗어난 아웃라이어 표시하지 않기 
		boxprops={'facecolor':'None'}, # 박스 색상 지우기 
```
