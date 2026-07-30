```cs
// 루트에 있는 Stage로 시작하는 이름의 게임 오브젝트를 찾음
GameObject stageObject = GetStageObject();

// 새로운 게임 오브젝트를 만들고, transform을 이용해 자식 오브젝트로 만든다. 
GameObject mapObject = new GameObject(MAP_OBJECT_NAME);
mapObject.transform.SetParent(stageObject.transform);


private GameObject GetStageObject()
{
	GameObject[] rootObjects = SceneManager.GetActiveScene().GetRootGameObjects();
	foreach (GameObject go in rootObjects)
	{
		if (go.name.StartsWith("Stage"))
		{
			return go;
		}
	}
	return null;
}
```

