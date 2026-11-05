- 크게 2가지로 나뉜다.

## 1. 업캐스팅(Upcasting)
- 자식 클래스 객체를 부모 클래스 타입으로 변환
- **항상 안전하며, 암시적으로 수행**된다.
```cs
Animal animal = new Dog();
```

## 2. 다운캐스팅(DownCasting)
- 부모 클래스의 객체를 자식 클래스 타입으로 변환
- **명시적으로 수행하며, 런타임 오류의 가능성**이 있다.
```cs
Dog dog = (dog)animal;
```

### 다운 캐스팅 안전하게 수행하기

#### 1. as 연산자
```cs
Dog dog = animal as Dog;
if (dog != null) 
{
	dog.Bark();
}
```

#### 2. is 연산자
```cs
if (animal is Dog) 
{
	Dog dog = (Dog) animal;
	dog.Bark();
}
```

#### 3. (C# 7.0 이상) 패턴 매칭
```cs
if (animal is Dog dog) 
{
	dog.Bark();
}
```

- 다형성을 활용하면서도 특정 상황에서 구체적인 기능을 사용할 때 유용하지만, 과도한 사용은 유연성을 해치기에 설계 단계에서 인터페이스나 추상 클래스를 적절히 활용하는 게 좋다.

