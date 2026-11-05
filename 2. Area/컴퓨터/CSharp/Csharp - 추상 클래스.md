```cs
abstract class Animal
{
    public abstract void MakeSound();

    public void Eat()
    {
        Console.WriteLine("동물이 먹이를 먹습니다.");
    }
}

class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("개가 짖습니다. 왈왈!");
    }
}
```

`abstract`로 선언하며, 추상 클래스라고 한다. 아래의 특징을 갖는다.

1. 추상 클래스는 인스턴스화할 수 없다. `new`로 인스턴스를 만들 수 없다.
2. 하나 이상의 추상 메서드를 포함할수 있다. 메서드 선언만 있고, 구현부는 없는 메서드이다.
3. 추상 클래스를 상속받는 파생 클래스는 모든 추상 메서드를 구현해야 한다. 이를 통해 `다형성Polymorphism`을 구현할 수 있다.
4. 추상 클래스는 일반 메서드, 속성, 필드 등을 포함할 수 있다.

**추상 클래스는 주로 공통적인 기능은 명시하지만 특정 메서드의 구현은 파생 클래스에 맡기려고 할 때 사용된다.**