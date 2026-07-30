- `foo.py`
```python
import optuna

def objective(trial):
    x = trial.suggest_float('x', -10, 10)
    return (x-2) ** 2
```

- 이걸 실행시키려면 커맨드라인에서 이런 입력을 할 수 있음
```
$ STUDY_NAME = `optuna create-study --storage sqlite:///example.db`
$ optuna study optimize foo.py objective --n-trials = 100 --storage sqlite:///example.db --study-name $STUDY-NAME
```
- 라고 사이트에는 나와 있는데 STUDY_NAME 부분부터 막힘;

## 커맨드 목록
-   `ask` : 새 trial을 만들고 파라미터를 제안
-   `best-trial` : 가장 좋은 trial을 보여줌
-   `best-trials` : `Pareto Front`에 위치한 trial들의 리스트를 보여줌
-   `create-study`
-   `delete-study` : 뒤에 study의 이름을 같이 지정해줘야 함
-   `storage upgrade` : 스키마 업그레이드
-   `studies` : study의 목록 보여줌
-   `study optimize` : 최적화 시작
-   `study set-user-attr` : 유저 속성을 보여줌
-   `tell` : ask로 시작된 trial을 끝냄
-   `trials` : trial들을 보여줌
