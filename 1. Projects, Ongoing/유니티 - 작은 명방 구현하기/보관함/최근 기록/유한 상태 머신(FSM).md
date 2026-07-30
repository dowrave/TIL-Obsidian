
## 유한 상태 머신이란?
- `Finite State Machine(FSM)`
- **현재 하나의 상태만을 가지며, 조건에 따라 다른 상태로 전환된다.**
- 현재는 반드시 하나의 상태만을 가져야 한다. 

### FSM 여부에 따른 차이
- 쓰지 않았을 때 
```cs
void Update()
{
    if (hp <= 0)
    {
        Die();
        return;
    }

    if (player == null)
    {
        Patrol();
    }

    if (player != null)
    {
        Chase();
    }

    if (distance < attackRange)
    {
        Attack();
    }
}
```
> 각각이 어떤 의미를 가진 상태인지에 대해서도 생각해야 함. 
### 상태(State)
- **현재 객체가 무엇을 하고 있는가?**
- 예를 들어 `Idle, Move, Attack, Dead` 등이 있음. 
- 그리고 **각 상태는 `상태에 들어올 때, 상태 중일 때, 상태에서 나갈 때` 해야 하는 일이 다르다.** 
	- 대부분 **`Enter, Update, Exit`라는 3가지 함수로 구현**한다.

- 예를 들어 `Move -> Attack`이 된다면 아래처럼 처리한다.
```
Move.Exit()
Attack.Enter()
Attack.Update()
Attack.Update()
Attack.Exit()
```

### 상태 전환 함수(Change State)
- FSM에서 가장 중요한 함수.
```cs
ChangeState(new AttackState());
```

위처럼 구현한다면, 내부적으로는 이런 처리가 된다.
```
현재 상태 Exit() -> 새 상태 생성 -> 새 상태 Enter() -> 새 상태 Update() -> ...
```

### FSM의 구성 요소
- **객체는 자신이 어떤 상태인지만 갖고 있다.**
```
Enemy.StateMachine.CurrentState
```

- 그리고 실제 행동은 각 상태가 결정한다.
```
IdleState
MoveState
AttackState
DeadState
```
```