- **어떤 이름을 가진 오브젝트를 찾고 싶을 때 사용**한다.
- 기본적으로 **호출하는 `transform(=게임 오브젝트)`의 1단계 자식들까지만 검사함**
	- 2단계 이상의 자식을 검사하고 싶다면, `transform.Find("child1/child2/target")` 같은 식으로 지정할 수 있음.
```cs
transform.Find("target")
transform.Find("child1/child2/target")
```