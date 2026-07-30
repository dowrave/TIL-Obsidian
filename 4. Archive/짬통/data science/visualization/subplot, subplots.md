- 그래프 분할할 때 사용 (아래 2개의 실행 결과는 동일함)


##### 1. plt.subplot()
```python
fig = plt.figure()

ax1= fig.add_subplot(2, 2, 1)
ax2= fig.add_subplot(2, 2, 2)
ax3= fig.add_subplot(2, 2, 3)
ax4= fig.add_subplot(2, 2, 4)

ax1.plot(dataset_1['x'], dataset_1['y'], 'o')
ax2.plot(dataset_2['x'], dataset_2['y'], 'o')
ax3.plot(dataset_3['x'], dataset_3['y'], 'o')
ax4.plot(dataset_4['x'], dataset_4['y'], 'o')
```

#### 2. plt.subplots()
```python
fig, ax = plt.subplots(2, 2)

ax[0, 0].plot(dataset_1['x'], dataset_1['y'], 'o')
ax[0, 1].plot(dataset_2['x'], dataset_2['y'], 'o')
ax[1, 0].plot(dataset_3['x'], dataset_3['y'], 'o')
ax[1, 1].plot(dataset_4['x'], dataset_4['y'], 'o')
```
- 여러 파라미터
```python
fig, ax = plt.subplots(2, 2, 
					  figsize = (15, 5),
					  sharey = True)
```
