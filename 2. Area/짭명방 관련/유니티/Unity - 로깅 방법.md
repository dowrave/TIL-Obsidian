> [!note]
> `Debug.Log()`는 아래 작업들을 처리한다.
> 1. 문자열 처리
> 2. 스택 추적 - **특히 비용이 큼**
> 3. I/O 작업
> 4. 에디터 오버헤드
> 
> - 그래서 `Debug.Log()`는 최종 빌드 버전에서는 웬만해선 빼는 게 좋음
> 1. `#if UNITY_EDITOR ~ #endif` 내에 디버깅 로그를 넣음
> 2. 더 좋은 방법은 커스텀 로거를 만드는 것

- 커스텀 로거 예시
```cs
public static class MyLogger
{
    // [System.Diagnostics.Conditional("UNITY_EDITOR")]
    // 이 속성은 "UNITY_EDITOR" 심볼이 정의되었을 때만 이 메소드 호출 코드를 컴파일에 포함시킵니다.
    // 즉, 빌드 시에는 MyLogger.Log(...) 호출 자체가 사라집니다. #if보다 훨씬 깔끔합니다.
    [System.Diagnostics.Conditional("UNITY_EDITOR")]
    public static void Log(object message)
    {
        Debug.Log(message);
    }

    [System.Diagnostics.Conditional("UNITY_EDITOR")]
    public static void LogWarning(object message)
    {
        Debug.LogWarning(message);
    }

    [System.Diagnostics.Conditional("UNITY_EDITOR")]
    public static void LogError(object message)
    {
        Debug.LogError(message);
    }
    
    // 에러 로그는 빌드에서도 보고 싶을 경우, 속성을 제거하면 됩니다.
    // public static void LogError(object message)
    // {
    //     Debug.LogError(message);
    // }
}

// 사용 예시
MyLogger.Log("플레이어가 점프했습니다.");
```

