- `Optuna`에서 병렬처리 하는 과정
	1. RDB 서버 켜기(PostgreSQL, MySQL 추천)
	2. `-storage`인자를 넣어 `study`를 만듦
	3. `study`를 여러 노드, 프로세스로 나눔
	-   위 과정은 [쿠버네티스 예제](https://github.com/optuna/optuna-examples/tree/main/kubernetes)로도 할 수 있음

1. Python cmd창에서..
```
$ mysql -u root -e "CREATE DATABASE IF NOT EXISTS example"
$ optuna create-study --study-name 'distributed-example' --storage 'mysql://root@localhost/example'
```

2. Optimization Script를 짬 : `foo.py`
```python
import optuna
import optuna.trial.Trial as trial

def objective(trial):
	x = trial.suggest_float('x', -10, 10)
	return (x-2) ** 2

if __name__ == "__main__":
	study = optuna.load_study(
							  study_name = 'distribute-example',
							  storage = 'mysql://root@localhost/example'
	)
	study.optimize(objective, n_trials == 100)
```

3. 여러 터미널을 띄운 뒤 `python foo.py`입력 -> 병렬처리

#### 참고사항
1. `foo.py`에 들어가는  `n_trials`는 각 터미널에서 실행되는 `trial`의 수를 의미함 
2. `SQLite`는 성능 상의 이유로 추천하지 않음
