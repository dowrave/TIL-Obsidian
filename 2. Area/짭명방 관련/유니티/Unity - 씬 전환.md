- 비동기 씬 로딩 중
```cs
while (asyncLoad.progress < 0.9f)
    {
        float progress = Mathf.Clamp01(asyncLoad.progress / 0.9f);
        loadingScreen?.UpdateProgress(progress * 0.5f); // 전체 로딩의 50%
        yield return null;
    }
```

에서 `0.9f`라는 특정 값을 쓰는 이유가 궁금해짐

## asyncLoad.progress는 0.9f에서 멈춤

### 1. 로딩 단계(0 ~ 0.9)
- 씬에 필요한 모든 애셋을 디스크 -> 메모리로 읽어들임
- 게임 오브젝트, 컴포넌트를 생성할 준비를 함
- 현재 실행 중인 씬에 영향을 주지 않으며 백그라운드에서 수행됨

### 2. 활성화 단계(0.9 ~ 1)
- 로딩된 씬을 현재 활성 씬으로 만듦
- 이전 씬의 오브젝트를 파괴하고, 새 씬의 모든 오브젝트에 대해 `Awake, OnEnable` 등의 초기화 메서드를 호출함
- 메인 스레드에서만 수행할 수 있다. 씬의 복잡도에 따라 순간적인 멈춤이나 끊김 현상을 유발할 수 있다.

