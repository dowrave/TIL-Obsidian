```cs
Tile[] allTiles = currentMap.GetComponentsInChildren<Tile>();
```

위의 코드가 정확히 어떤 역할을 수행하는지 헷갈려서 정리해둔다.
현재 하이어라키의 구조는 대충 아래와 같다.
```
Stage
- Map
	- Tile 1 (tile 스크립트를 가짐)
		- Cube
	- Tile 2 (tile 스크립트를 가짐)
		- Cube
```

1. **GetComponentsInChildren**
Tile 스크립트를 컴포넌트로 가진, 게임 오브젝트들의 해당 Tile 컴포넌트에 대한 참조를 반환한다. 예를 들면 Map 안에 Tile 오브젝트가 있고, Tile 오브젝트가 Tile 스크립트를 가진다고 하면 해당 스크립트(=컴포넌트)들의 참조를 반환한다.

2. 이 때, Tile 게임 오브젝트는 아래의 코드로 접근할 수 있다.
```cs
GameObject tileObject = tile.gameObject;
```

3. 그리고 게임 오브젝트 참조를 통해, 해당 게임 오브젝트의 다른 컴포넌트들에 접근할 수 있다.
```cs
GameObject tileObject = tile.gameObject;

Render tileRenderer = tile.GetComponent<Renderer>();

// 자식 오브젝트 Cube에 접근하기
Transform cubeTransform = tile.transform.Find("Cube");
```

> 자식 오브젝트로 접근하는 방법으로는 
> 1. 위에서 나온 `gameObject.GetComponentsInChildren<Component>().gameObject;`
> 2. `gameObject.transform.Find("Object Name")`


이렇게 **Tile 스크립트(컴포넌트)를 통해 게임 오브젝트와 다른 컴포넌트, 심지어는 자식 컴포넌트까지도 접근할 수 있다**는 게 유니티의 `컴포넌트 기반 아키텍처`의 강력한 특징 중 하나이다.