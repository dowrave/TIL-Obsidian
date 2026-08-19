#Csharp 

## switch 문(Statement)
```cs
// switch 문 (기존)
int result;
switch (c)
{
    case '+': result = a + b; break;
    case '-': result = a - b; break;
    default: throw new InvalidOperationException();
}
```

## switch 식(Expression)
```cs
// switch 식 (C# 8+)
int result = c switch
{
    '+' => a + b,
    '-' => a - b,
    _ => throw new InvalidOperationException($"알 수 없는 연산자: {c}")
};
```
- "식"은 그 자체로 값을 만들어내서 바로 대입하거나(`stack.Push` 등에 바로 사용할 수 있음) 인자로 쓸 수 있다. 
- `case` / `break` 대신 `=>`으로 값을 바로 연결하고, `default` 대신 `_(discard)`를 쓴다.
- `break`을 빠뜨릴 일이 없어서 실수도 줄어든다. 

>[!question]
>- `switch` 식expression은 문statement을 완전히 대체할 수 있는가?
- **아니다.** 용도가 다르다고 보는 게 적합함.

- **`switch 식`은 값을 만들어서 바로 쓰고 싶을 때를 위한 것**이다. 대입, `return`, 인자 전달 등의 표현식이 와야 하는 자리에 쓴다.

- **`switch 문`은 여러 줄의 동작을 실행하고 싶을 때를 위한 것이다. 꼭 값을 만들 필요는 없다.**

```cs
// 이런 건 "식"으로 대체할 수 없음 
// 식은 분기당 하나의 표현식만 허용한다.
case '+':
      Console.WriteLine("더하기");
      result = a + b;
      break;
```

- 따라서 "편의성"을 개선하기 위해 '식'이 나왔다기보다는, 표현식 자리에서 값을 안전하게 만든다는 목적에 가깝다. 