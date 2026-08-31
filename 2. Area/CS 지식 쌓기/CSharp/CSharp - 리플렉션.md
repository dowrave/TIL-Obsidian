#Csharp 

실행 중인 프로그램이 자기 자신의 타입 정보(클래스, 필드, 프로퍼티 ,메서드 등)를 코드로 들여다보고 조작할 수 있게 해주는 기능.

- 보통 C# 코드를 작성할 때는, 객체의 타입과 객체가 가진 필드를 컴파일 시점에 이미 다 안다.
```cs
var data = new { value = 5 };
int v = data.value; // 컴파일러는 data에 value 필드가 있다는 걸 컴파일 시점에 이미 앎
```

- 그런데 리플렉션을 쓰면 타입을 몰라도(코드 작성 시점에 존재하지 않았던 타입이더라도) 그 객체를 실행 중에 조사해서 안에 뭐가 들었는지를 알 수 있다.
```cs
object data = new { value = 5 };

// 컴파일 시점에 data가 무슨 타입인지 모른다고 가정.
// 런타임에 value라는 이름의 프로퍼티가 있는지를 직접 물어본다.
var property = data.GetType().GetProperty("value");
object result = property.GetValue(data); // 5
```

## 리플렉션 핵심 타입 / 네임스페이스
```cs
using System.Reflection;
```

1. `object.GetType()` : 모든 객체가 상속받는 메서드로, 실제 런타임 타입 정보`System.Type 객체`를 돌려준다.
2. `Type.GetProperty(string name), GetProperties()`
	- 해당 타입이 가진 프로퍼티 정보`PropertyInfo`를 이름으로 찾거나 전부 가져온다. 
3. `PropertyInfo.GetValuie(object instance)`
	- 찾은 프로퍼티 정보를 바탕으로 실제 인스턴스 `data`에서 그 값을 꺼낸다.

- 리플렉션이 아닌 것과 비교
```cs
var data = new { value = 5 };
int a = data.value;              // 일반 접근 - 컴파일러가 컴파일 시점에 이미 알고 처리
int b = (int)data.GetType().GetProperty("value").GetValue(data); // 리플렉션 - 런타임에 이름으로 찾아서 접근
```


## 장단점
- 장점 : 타입을 몰라도 다룰 수 있다. 범용 API를 만들 때 유용하다.
- 단점 : 일반적인 코드 실행보다 느리다(`data.value` 같은 직접 접근과 비교)

## Smart Format에서 이걸 쓰는 이유
- `new { value = index + 1 }`처럼 익명 타입의 객체를 `Smart Format`에 넘기면 해당 라이브러리 입장에서는 무슨 타입인지 알 방법이 없다. 익명 타입이 전부 다르기 때문이다. 
- 그래서 라이브러리 내부에서는
	1. 넘어온 객체의 타입을 리플렉션으로 조사
	2. `"value"`라는 이름의 프로퍼티가 있는지 찾음
	3. 있으면 값을 꺼내 `{value}` 자리에 채워넣는다.

