```cs
    protected void SetTransparentMaterial()
    {
        previewMaterial.SetFloat("_Mode", 3); // TransParent 모드로 설정
        previewMaterial.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.SrcAlpha); // 소스 블렌딩 모드 설정. 알파값을 사용해 블렌딩
        previewMaterial.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.OneMinusSrcAlpha); // 대상 블렌딩 모드 설정. (1 - 소스 알파값)으로 블렌딩
        previewMaterial.SetInt("_ZWrite", 0); // Z버퍼 비활성화. 투명 객체는 사용하지 않는다고 함
        previewMaterial.DisableKeyword("_ALPHATEST_ON"); // 알파 테스트 모드 비활성화
        previewMaterial.EnableKeyword("_ALPHABLEND_ON"); // 알파 블렌딩 모드 활성화
        previewMaterial.DisableKeyword("_ALPHAPREMULTIPLY_ON"); //알파 프리멀티플라이 모드 비활성화
        previewMaterial.renderQueue = 3000; // 렌더 큐 설정. 불투명 객체들 뒤에 그려지게 함
    }
```

> 프로젝트 중 투명 머티리얼을 설정하는 과정에서 AI가 이런 코드를 뱉었다. 하나하나 가볍게 알아본다. 컴퓨터 그래픽스와 렌더링 파이프라인의 중요한 부분이래요.

## 블렌딩 모드
```cs
previewMaterial.SetInt("_SrcBlend", (int)UnityEngine.Rendering.BlendMode.SrcAlpha); // 소스 블렌딩 모드 설정. 
previewMaterial.SetInt("_DstBlend", (int)UnityEngine.Rendering.BlendMode.OneMinusSrcAlpha); // 대상 블렌딩 모드 설정. 
```
- 새로 그려질 픽셀(`소스`)과 이미 화면에 있는 픽셀(`대상`)을 어떻게 합칠지 결정함
- `SrcAlpha` : 소스 픽셀의 알파값 사용
- `OneMinusSrcAlpha` : `1 - 소스 알파값`
- 위 둘을 함께 사용하면 전형적인 알파 블렌딩이 된다.

## Z 버퍼(Depth Buffer)
```cs
previewMaterial.SetInt("_ZWrite", 0);
```
- 3D 공간에서 객체의 깊이 정보를 저장하는 버퍼.
- `Z-Write`를 끄면 해당 객체가 다른 객체를 가리지 않는다. 투명 객체에서 사용한다.

## 알파 테스트 모드
```cs
previewMaterial.DisableKeyword("_ALPHATEST_ON");
```
- 활성화 시, 픽셀의 알파값이 특정 임계값 이하면 그리지 않는다.
- 나뭇잎, 철망 등에 사용된다.

## 알파 블렌딩 모드
```cs
previewMaterial.EnableKeyword("_ALPHABLEND_ON");
```
- 픽셀의 알파값에 따라 배경과 부드럽게 섞는다.
- 반투명한 유리, 연기 등의 효과에 사용된다.

## 알파 프리멀티플라이 모드
```cs
previewMaterial.DisableKeyword("_ALPHAPREMULTIPLY_ON");
```
- RGB값에 알파값을 미리 곱해 저장한다.
- 일부 상황에서 더 정확한 블렌딩 결과를 제공한다.

## 렌더 큐
```cs
previewMaterial.renderQueue = 3000;
```
- 객체들이 그려지는 순서를 결정한다.
- 값이 낮을수록 먼저 그려진다.
- 불투명 객체는 2000, 투명 객체는 3000으로 설정한다.

일반적으로는 유니티의 기본 설정이나 머티리얼 인스펙터 등으로 조정하지만, 특별한 효과나 최적화가 필요한 경우 이런 식으로 코드로 구현할 수 있다.