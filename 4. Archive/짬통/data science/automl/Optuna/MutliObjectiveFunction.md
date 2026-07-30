- 코드가 길고 `Pytorch`라 한눈에 알 수는 없음
- 중요한 포인트만 짚고 넘어감

1. 여러 목적함수를 정의한 방법
```python
def eval_model(model, valid_loader):
    model.eval()
    correct = 0
    with torch.no_grad():
        for batch_idx, (data, target) in enumerate(valid_loader):
            data, target = data.view(-1, 28 * 28).to(DEVICE), target.to(DEVICE)
            pred = model(data).argmax(dim=1, keepdim=True)
            correct += pred.eq(target.view_as(pred)).sum().item()

    accuracy = correct / [N_VALID_EXAMPLES]

    flops, _ = thop.profile(model, inputs=(torch.randn(1, 28 * 28).to(DEVICE),), verbose=False)
    return flops, accuracy

```
- 모델 평가 항목에서 `accuracy`와 `flops`를 정의 했음
- 그리고 `objective(trial)`에서.. 
```python
def objective(trial):
""" .... """
    model = define_model(trial).to(DEVICE)

    optimizer = torch.optim.Adam(
        model.parameters(), trial.suggest_float("lr", 1e-5, 1e-1, log=True)
    )

    for epoch in range(10):
        train_model(model, optimizer, train_loader)
    flops, accuracy = eval_model(model, val_loader)
    return flops, accuracy
```
- 이렇게 정의한 다음
```python
study = optuna.create_study(directions=["minimize", "maximize"])
study.optimize(objective, n_trials=30, timeout=300)
```
- 각각의 목적 함수에 대해 방향을 정해주면 됨
	- `FLOPS` : 계산 수이므로 최소화
	- `accuracy` : 최대화

2. 다목적함수 시각화
```python
optuna.visualization.plot_pareto_front(study, target_names=["FLOPS", "accuracy"])
```
- `pareto_front` : 다른 하나의 지표를 희생하지 않고 어떤 지표를 더 이상 올릴 수 없는 지점
	- 즉 위의 하이퍼파라미터 탐색으로 나타난 최적의 지점 이상으로 어떤 지표에 대한 성능을 개선하고 싶다면, 다른 지표에 대한 성능은 희생해야 함
- 참고 ) `pareto_front`는 목적함수가 `2개, 3개`인 경우에만 사용가능하다!