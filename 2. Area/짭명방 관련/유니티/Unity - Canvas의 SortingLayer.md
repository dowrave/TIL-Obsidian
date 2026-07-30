UI 요소 간의 순서를 제어하려면, **Canvas의 Sorting Order**를 설정하여 어떤 UI가 다른 UI 위에 렌더링될지를 제어할 수 있습니다.

1. **Canvas** 컴포넌트에서 `Sorting Layer`와 `Order in Layer`를 사용하여 UI의 렌더링 순서를 조정합니다.
    
    - **Sorting Layer**: 유니티의 렌더링 순서를 정의하는 레이어입니다. 기본적으로 모든 UI는 `Default` Sorting Layer에 속합니다.
    - **Order in Layer**: 동일한 `Sorting Layer` 내에서 렌더 순서를 지정합니다. 숫자가 클수록 앞에 렌더링됩니다.
2. **Canvas**가 없는 UI 요소라면, 부모 `Canvas`의 `Sorting Order`에 따라 순서가 결정됩니다.