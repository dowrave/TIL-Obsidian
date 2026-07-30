#Unity 

## Attribute(속성)란?
- 대괄호`[]`로 감싸진 기능들. `SerializeField`가 대표적인 예시.
- 코드 자체의 로직에 관여하진 않으나, **컴파일러나 유니티 엔진에게 "이 부분은 특별하게 다뤄달라"고 전달**하는 역할을 한다.

### 예시 
- `[SerializeField]` : 유니티에게 `private` 변수임에도 인스펙터에 띄우라는 메모
- `[Range(0, 10)]` : 유니티에게 변수를 슬라이더로 0 ~ 10으로 조절할 수 있게 하라는 메모
- `[System.Serializable]` : 클래스를 데이터로 저장할 수 있게 직렬화하라는 메모



## 새로운 Attribute 정의 및 사용 예시

### 상황
- 유니티 6.3버전 기준 사용하는 C#은 9.0버전이다.
- 아래 코드는 상태와 값을 동시에 출력시키기 위한 코드.
```cs
// 특정 필드의 이름과 값 확인하는 메서드
[System.Diagnostics.Conditional("UNITY_EDITOR")]
public static void LogFieldStatus(object message, [CallerArgumentExpression("message")] string name = "")
{
	Debug.Log($"{name} : {message}");
}
```

여기서 `CallerArgumentExpression("message")`는 이 자체로 사용할 수 없다. 
컴파일러`Roslyn`은 최신 버전이라 이 기능을 아는데, **`.NET 라이브러리`에서는 해당 기능의 이름표`Attribute`가 포함되어 있지 않기 때문이다.**

따라서 이를 구현하기 위한 코드를 추가해줘야 한다. 
어디에 추가해도 상관은 없음. `Attributes.cs`에 구현했다.

```cs
namespace System.Runtime.CompilerServices
{
    // CallerArgumentExpression("message")를 쓰기 위한 정의
    [AttributeUsage(AttributeTargets.Parameter, AllowMultiple = false, Inherited = false)]
    internal sealed class CallerArgumentExpressionAttribute : Attribute
    {
        public CallerArgumentExpressionAttribute(string parameterName)
        {
            ParameterName = parameterName;
        }
        public string ParameterName { get; }
    }
}
```

### 위 코드의 의미
- 큰 의미는 `CallerArgumentExpression` 사용 방법에 대한 정의이다.

### 기술적인 원리
- 프로그래밍 용어로 `폴리필Polyfill`이라고 부른다.
	- 구형 환경에서 신기술을 쓰기 위해 구멍을 메우는 행위.
	- 구멍 : `CallerArgumentExpression`이라는 기능이 없었음.
	- 구멍 메움 : 기능을 수동으로 구현했음. 컴파일러가 이를 인식해서 정상적으로 작동.
	- 여기서 구현한 기능은 `[CallerArgumentExpression]`이라는 어트리뷰트임. 이 어트리뷰트는 라이브러리 내부에 있는 기능과 연결시키는 역할을 해줌. 이거 자체가 변수를 변수 자체의 이름값으로 바꿔주는 게 아님.

1. **컴파일러의 동작 방식** : `C# 컴파일러`가 `[CallerArgumetnExpression]` 코드를 만나면, 내부적인 규칙에 따라 `System.Runtime.CompilerServices` 네임스페이스 내부의 `CallerArgumentExpressionAttribute` 클래스를 찾는다.
2. **존재 여부 확인** : 원래는 마이크로소프트의 동적 라이브러리 내에 클래스가 있어야 하나, 유니티의 `.NET` 버전에는 포함되지 않았다.
3. **사용자 정의 클래스** : 똑같은 이름, 네임 스페이스로 클래스를 선언하면 컴파일러가 해당 클래스를 가져다 쓴다.
4. 기능 수행 : 클래스 자체에는 아무 기능이 없다. **컴파일러에게 "이 파라미터에는 변수 이름을 문자로 넣어줘"라는 표식 역할을 한다.** 변수 이름을 텍스트로 바꾸는 작업은 컴파일러가 처리한다. 

### 어트리뷰트 상세 분석
```cs
// 컴팡일러가 기능을 찾기 위해 찾는 네임스페이스
namespace System.Runtime.CompilerServices
{
	// 이 속성Attribute을 어디에 붙일 수 있는지 설정한다.
	// Parameter에만 붙일 수 있다고 설정한다.
	// 세부적인 설정은 MS의 그것과 동일한 것임
	// AllowMultiple = false : 동시에 하나의 이름만 사용 가능하게 하기
	// Inherited = false : 상속되지 않게 하기. MS의 원본 규격 설정이기도 하다.
    [AttributeUsage(AttributeTargets.Parameter, AllowMultiple = false, Inherited = false)]
    
    // internal : 이 프로젝트 내부에서만 사용한다 
    // 유니티 버전이 업데이트돼서 진짜 공식 클래스가 생겼을 때의 충돌을 방지하기 위해 public이 아닌 internal을 쓴다.
    internal sealed class CallerArgumentExpressionAttribute : Attribute
    {
    
	    // 어떤 매개변수를 가져올지 지정하는 생성자.
	    // [CallerArgumentExpression("message")] : "message"라는 매개변수의 식을 가져오라는 의미
        public CallerArgumentExpressionAttribute(string parameterName)
        {
            ParameterName = parameterName;
        }
        public string ParameterName { get; }
    }
}
```
> 여기서 주의해야 할 점은 **"변수의 이름을 텍스트로 바꾸는 로직"은 정의하지 않았다**는 것이다. 그건 `CallerArgumentExpression`이 자동으로 처리함


### 어트리뷰트 사용 지점 상세 분석
```cs
    [System.Diagnostics.Conditional("UNITY_EDITOR")]
    public static void LogFieldStatus(object message, [CallerArgumentExpression("message")] string name = "")
    {
        // Debug.LogError(message);
        Debug.Log($"{name} : {message}");
    }
    

// 실사용
int number = 1;
LogFieldStatus(number); // number: "1"
```
> - 위의 정의에 따라, `name` 필드에 들어가는 `string` 값은 `message`에 들어가는 변수 이름 값으로 설정된다.

---
## 그래도 헷갈리면

- 이미 구현된 기능
	- 변수를 문자열로 바꾸는 기능
	- 파라미터에 적용하는 기능
	- `CallerArgumentExpression`이라는 어트리뷰트가 붙어야만 사용 가능

- 지금 `Attribute`를 추가하면서 새로 생긴 기능
	- `class CallerArgumentExpressionAttribute`을 만들어줌 - 즉 **어떤 기능을 만든 게 아니라 그 기능에 연결해주는 다리를 만들었다라고 생각하면 쉬움**


