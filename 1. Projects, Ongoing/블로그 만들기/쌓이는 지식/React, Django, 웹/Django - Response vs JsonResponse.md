### Response vs JsonResponse

- **Response (Django Rest Framework)**:
    
    - DRF의 `Response` 클래스는 JSON 뿐만 아니라 다양한 컨텐트 타입을 지원합니다. 클라이언트의 요구 사항에 따라 JSON, HTML, XML 등 다양한 형식으로 응답을 반환할 수 있습니다.
    - `Response` 객체는 일반적으로 DRF와 함께 사용되며, API 뷰에 적합합니다.
- **JsonResponse (Django)**:
    
    - Django의 `JsonResponse` 클래스는 JSON 데이터를 반환하는 데 특화되어 있습니다. 주로 Django의 기본 뷰에서 JSON 형식의 응답을 쉽게 만들기 위해 사용됩니다.
    - `JsonResponse`는 Django 기본 패키지의 일부이며, DRF를 사용하지 않는 경우에 주로 사용됩니다.

간단히 말하면, `Response`는 DRF의 일부로 더 유연하고 다양한 기능을 제공하는 반면, `JsonResponse`는 Django의 기본 기능으로 JSON 응답에 좀 더 특화되어 있습니다.