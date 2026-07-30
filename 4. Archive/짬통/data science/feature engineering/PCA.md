[깃허브 정리 내용](https://github.com/dowrave/TIL/blob/main/FeatureEngineering/5_PCA.ipynb)
- 클러스터링이 근접성으로 데이터를 나눴다면, **PCA는 분산에 따라 데이터를 나눈 것**이다. **PCA는 데이터 내의 중요한 관계와 유용한 특성을 찾을 수 있도록 도와준다.**
-   PCA는 표준화된 데이터를 적용한다.
    -   표준화된 데이터에서 `Variation`은 `Correlation`을 의미한다.
    -   표준화되지 않은 데이터에서 `Variation`은 `Covariance`를 의미한다.

- **PCA의 핵심은, 데이터를 원래 축으로 설명하는 대신 변동 축으로 설명하는 것이다.**
	- 새로운 축들은 원래 축들의 선형 결합으로 생각할 수 있다.
		- 이를 수식으로 나타낼 때 원래 축 앞에 붙는 계수를 `Loading`이라고 한다.
		- 새로운 축을 `주성분 Principle Component`이라고 한다.

- **PCA는 각 성분의 분산(Variance)을 알려주기도 한다.** 길게 걸쳐있는 축에 대한 분산이 제일 큼
- 또한 **Percent Of Experienced Variance라는 값을 통해 각 주성분이 분산에 얼마나 기여하는지도 볼 수 있다.**
- 앞에서의 내용과 마찬가지로, **분산값이 예측 변수로서 우수한지를 반드시 나타내는 건 아니다 : 예측하려는 내용에 따라 다르다.**


### 특성 공학을 위한 PCA
1.  **성분이 분산을 알려주므로, 성분에 대한 MI 점수를 확인한 다음 목표값을 예측할 수 있는 변동의 종류를 확인할 수 있다.** 점수가 높은 요소에 대한 클러스터링을 시도할 수도 있고, 새로운 특성을 만들어내는 아이디어로 쓸 수도 있다.
2.  **성분 자체를 특성으로 사용할 수 있다.** 데이터의 분산을 가장 잘 나타내기 때문에 원래 특성보다 유용할 수 있다.
    -   차원 축소 : PCA는 중복된 기능 중 하나 이상을 반드시 0에 가까운 성분으로 분할한다. 이러한 성분을 자연스럽게 지울 수 있다.
    -   이상 감지 : 분명하지 않은 비정상적 변동은 종종 저분산 성분에서 나타난다. 이상 징후나 특이치 감지 작업에서 유용한 경우가 많다.
    -   노이즈 감소 : 센서는 노이즈를 받아들이곤 하는데, PCA를 통해 노이즈를 유지하면서 더 유용한 정보를 수집할 수 있다. 따라서 신호 : 잡음 비율을 높일 수 있다.
    -   상관성 분해 : PCA는 상관성(Correlation)이 있는 기능을 상관성이 없는 요소로 분할하므로, 알고리즘 작업을 더 쉽게 할 수 있다.

### 팁
- PCA는 **연속 변수에만 사용가능**하고, **스케일링에 민감**하며, 이상치는 결과에 과한 영향을 줄 수 있음.

#### Python

```python
from sklearn.decomposition import PCA

pca = PCA()
X_pca = pca.fit_transform(X_scaled)

component_names = [f"PC{i+1}" for i in range(X_pca.shape[1])]
X_pca = pd.DataFrame(X_pca, columns = component_names)

# 중요한 건 여기 : 각 주성분이 기존 특성들의 선형 결합으로 어떻게 이뤄졌는가를 볼 수 있음
loading = pd.DataFrame(
                        pca.components_.T, # loading 행렬 Transpose
                        columns = component_names, # 각 열은 주성분
                        index = X.columns # row는 원래 피쳐들
)
```

- `MI Score`와 결합할 수도 있음
```python
mi_scores = mutual_info_regression(X_pca, y, discrete_features = False)
mi_scores = pd.Series(mi_scores, name = 'MI Scores', index = X.columns)
mi_scores = mi_scores.sort_values(ascending = False)
```

- 예제에서는 `PC3`이 `horsepower`와 `curb_weight` 간의 `loading`의 크기가 비슷하고 방향이 반대인 걸 보고는 `ratio` 특성을 만든 뒤 2차 회귀선을 살펴봤다.

#### 활용
- PCA를 통해 데이터를 보는 것에서 그치지 않고, 이를 통해 새로운 특성을 만들 수 있어야 함
- 예제에서도 위에서 언급한 내용처럼

1. MI 스코어를 활용해 새로운 feature를 만듦
	-   MI 스코어에서 좋은 값을 지니는 주성분이 있을 거임
	-   또한 각 주성분의 구성은 `Loading` 행렬에서 어떤 피쳐가 어떤 가중치로 들어갔는지 볼 수 있음
	-   이 특성들을 선형결합(`+`나 `*`)을 통해 새로운 feature로 만들어주는 방식임
```python
X["Feature1"] = X.GrLivArea + X.TotalBsmtSF 
X["Feature2"] = X.YearRemodAdd * X.TotalBsmtSF
```
2.  `pca.fit_transform()`으로 나온 행렬을 그대로 새로운 특성으로 집어넣어줌
```python
X = X.join(X_pca)
```

- 이외에도 이상치 탐색도 가능하다 
	- 각 주성분과 타겟값에 대해 박스 플롯을 그렸을 때 나타나는 이상치들이 있다는 거임
	- 예제에서 설명한 내용을 직접 눈으로 파악하긴 어려웠다.
