#Csharp #Unity 

## 예시
```cs
public abstract class UnitSkill<Tcaster>: ScriptableObject where Tcaster: UnitEntity
```

라는 가장 부모 클래스가 있을 때

`UnitEntity`의 자식인 `Operator`에 대해 정의된 `UnitSkill<Operator>`의 자식 클래스에 있는 아래 메서드는 

```cs
public StatModificationBuff(float duration, StatModifierSkill.StatModifiers mods, UnitSkill<UnitEntity> baseSkill)
{
	this.duration = duration;
	buffName = "Stat Boost";
	modifiers = mods;
	SourceSkill = baseSkill;
}
```

여기서 `UnitSkill<UnitEntity>`를 받아 초기화하는 것이 불가능함.

>![note]
>- C#에서 `UnitSkill<Operator>`는 `UnitSkill<UnitEntity>`로 암묵적 변환이 불가능함


## 해결법
- `UnitSkill<Tcaster>` 위에 있는 클래스를 하나 추가해주면 됨.
```cs
// 비제네릭 베이스. 내용물을 비워도 된다.
public abstract class UnitSkillBase : ScriptableObject { }

public abstract class UnitSkill<Tcaster>: UnitSkillBase where Tcaster: UnitEntity
{
	// ...
}

public StatModificationBuff(float duration, StatModifierSkill.StatModifiers mods, UnitSkillBase baseSkill)
{
	this.duration = duration;
	buffName = "Stat Boost";
	modifiers = mods;
	SourceSkill = baseSkill;
}
```