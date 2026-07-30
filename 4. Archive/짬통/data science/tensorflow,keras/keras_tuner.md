[공식 문서](https://keras.io/api/keras_tuner/)
- 하이퍼 파라미터 탐색에 이용함
```python
import keras_tuner

def build_model(hp):
  model = keras.Sequential()
  model.add(keras.layers.Dense(
						      hp.Choice('units', [8, 16, 32]), 
						      activation='relu'))
  model.add(keras.layers.Dense(1, activation='relu'))
  model.compile(loss='mse')
  return model

# hp를 전달하는 곳은 여기다
tuner = keras_tuner.RandomSearch(
    build_model,
    objective='val_loss',
    max_trials=5,
    overwrite = True                 # 중요!
    )

tuner.search(x_train, y_train, epochs=5, validation_data=(x_val, y_val))
best_model = tuner.get_best_models()[0]
```
- `RandomSearch` 외에도 `BayesianOptimization`, `Hyperband` 도 있다. (요 2개는 공부 ㄱ)
	- 탐색 범위가 넓어질수록 `RandomSearch`의 경쟁력이 떨어진다.
	- `Hyperband` :   하이퍼파라미터 쌍의 개수를 임의로 선정한 뒤, 적은 에포크를 돌리면서 스코어가 낮은 하이퍼파라미터 쌍은 탈락시키고, 스코어가 높은 하이퍼파라미터는 에포크를 더 돌림
	
- **`overwrite = True`는 중요함** : 탐색 결과를 파일로 저장하는데, 동일한 이름의 파일이 있다면 훈련을 진행하지 않고 이미 훈련된 결과를 출력함

- 조건부 모델 구성
```python
class MyHyperModel(HyperModel):
    def build(self, hp):
        model = Sequential()
        model.add(Input(shape=(32, 32, 3)))
        model_type = hp.Choice("model_type", ["mlp", "cnn"])
        with hp.conditional_scope("model_type", ["mlp"]):
            if model_type == "mlp":
                model.add(Flatten())
                model.add(Dense(32, activation='relu'))
        with hp.conditional_scope("model_type", ["cnn"]):
            if model_type == "cnn":
                model.add(Conv2D(64, 3, activation='relu'))
                model.add(GlobalAveragePooling2D())
        model.add(Dense(10, activation='softmax'))
        return model

tuner = keras_tuner.RandomSearch(
    build_model,
    objective='val_loss',
    max_trials=5)
```