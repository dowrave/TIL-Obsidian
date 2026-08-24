#Csharp 

- **클래스의 인스턴스를 배열처럼 `객체[인덱스]` 문법으로 접근**할 수 있게 해주는 C#의 문법적 기능.

- 아래처럼 사용한다.
```cs
public class Inventory
{
	private string[] items = new string[10];
	
	// 인덱서 정의(this 키워드 사용)
	public string this[int index]
	{
		get { return items[index]; } // myInv[0] 실행 시 호출
		set { items[index] = value; } // myInv[0] = "검" 실행 시 호출
	}
}

// 사용 예시
Inventory myInv = new Inventory();
myInv[0] = "장검"; // set
Debug.Log(myInv[0]); // get
```

## 역사
- 2002년 C# 1.0에 처음 나왔을 때부터 있었다.

### 왜 만들었을까?
- C#의 수석 설계자 아더 헤일스버그는 기존 언어들의 장단점을 연구했다.
	- C++ 
		- `operator[]` 연산자 오버로딩을 통해 객체를 배열처럼 쓰게 해줬다. 
		- 그러나 문법이 복잡하고 제약이 적어서 오용되기 쉬웠다.
	- Java 
		- 연산자 오버로딩을 완전히 금지했다. 
		- 커스텀 컬렉션을 만들면 무조건 `list.get(i)`, `list.set(i, value)` 같은 메서드를 만들어 써야 해서 문법이 번거로웠다.

- 객체의 캡슐화를 지키는 자바의 장점을 유지하면서, C++의 배열처럼 `[]`로 깔끔하게 접근할 수 있는 방법을 고민했다.

### 해결책 : 매개변수를 갖는 프로퍼티 Indexer
- C#에는 이미 필드를 안전하게 감싸는 `프로퍼티(get, set)` 개념이 있었다. 
- 여기에 인덱스 매개변수를 결합해서, **`매개변수가 있는 프로퍼티`라는 개념으로 `인덱서`가 탄생**했다.
```cs
// Java 스타일 - 번거로움
myList.Set(0, "Item");
string item = myList.Get(0);

// C# 인덱서 스타일 - 직관적이고 안전
myList[0] = "Item";
string item = myList[0];
```

### 인덱서의 컴파일러 - 인터내셔널 호환성
- C# 컴파일러는 인덱서를 내부(IL 코드)적으로 `get_Item()`과 `set_Item()`이라는 메서드로 자동변환해서 컴파일한다. 
- .NET 언어 간의 호환성 때문으로, C#에서 만든 인덱서 클래스를 인덱서 문법이 없는 다른 .NET 언어(C++, CLI 등)에서 쓸 때 `get_Item()` 형태로 호출할 수 있게끔 설계된 개념이다.

