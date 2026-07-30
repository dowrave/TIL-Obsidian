- 이걸 쓸 일이 생기네
|     | track_id | velocity_km/h | race_distance_km/h |
| --- | --------- | ------------- | ------------------ |
| 0   | AQU       |     65          |  1.63                  |
| 1   | BEL       |       68        |      1.76              |
| 2   | SAR          |      67         |       1.76             |

- 위 데이터프레임을 `track_id`마다 2개의 막대 그래프를 그리고 싶다고 하자
- 일단 `sns.barplot(y = ['velocity', 'distance']`는 작동하지 않음
- 이 때 쓰는게 `df.melt()`이다.

```python
tidy = dist_speed.melt(id_vars = 'track_id')
```
<실행 결과>
|     | track_id | variable           | value |
| --- | -------- | ------------------ | ----- |
| 0   | AQU      | velocity_km/h      | 65    |
| 1   | BEL      | velocity_km/h      | 68    |
| 2   | SAR      | velocity_km/h      | 67    |
| 3   | AQU      | race_distance_km/h | 1.63  |
| 4   | BEL      | race_distance_km/h | 1.76  |
| 5   | SAR      | race_distance_km/h | 1.76  |

- 근데 스케일이 너무 다르다면 그냥 따로 표시하는 게 나아보인다. 
- 합쳐서 표시하면 너무 지저분한 느낌이 있기도 함

### 참고 : multi columns에 써도 잘 적용됨
- 가령 `(D, FT), (D, GD)`이런 식으로 column이 계층적으로 있을 때 `.melt()`를 쓰면 `D | FT`, `D | GD` 이런식으로 잘 분리됨.
- 