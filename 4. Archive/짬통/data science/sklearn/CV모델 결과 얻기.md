```python
model.cv_results_ # 에는 이런 정보들이 있다. 이외에도 문서에 많음
{
'mean_test_score'    : [0.81, 0.67, 0.70],
'mean_train_score'   : [0.81, 0.74, 0.70],
'std_train_score'    : [0.01, 0.19, 0.00],
'params'             : [{'kernel' : 'rbf', 'gamma' : 0.1}, ...],
}
```


---------------------------
##### DF로 옮기기
- 이것들은 판다스 데이터프레임으로 바로 만들 수 있음 `pd.DataFrame(model.cv_results_)
```python
temp = pd.DataFrame(model.cv_results_['params']).merge(pd.DataFrame(model.cv_results_																 ['mean_test_score'], columns = ['mean_test_score']), left_index = True, right_index = True)
```
- 파라미터 값들 + 평균 검증 스코어를 뽑아냄
