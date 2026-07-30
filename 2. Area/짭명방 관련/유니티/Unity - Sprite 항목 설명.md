#Unity 

![[Pasted image 20241028175315.png]]

1. Sprite Mode:
```plaintext
Single: 하나의 이미지를 하나의 스프라이트로 사용
Multiple: 하나의 이미지를 여러 스프라이트로 분할 (스프라이트 시트/아틀라스)
Polygon: 불규칙한 모양의 스프라이트 (투명 영역 최적화)
```

2. 기본 스프라이트 설정:
```plaintext
Pixels Per Unit: 
- 스프라이트의 픽셀과 유니티 월드 유닛 간의 비율
- 100이면 100픽셀 = 1유니티 유닛
- UI의 경우 보통 100 사용

Mesh Type:
- Full Rect: 전체를 사각형 메시로 생성
- Tight: 알파 영역을 제외한 실제 이미지 영역만 메시로 생성
- (메모리/성능 최적화에 영향)

Extrude Edges: 
- 스프라이트 가장자리 확장 픽셀 수
- 필터링 시 가장자리 아티팩트 방지용

Pivot: 
- 스프라이트의 중심점 위치 설정
- 회전/스케일링의 기준점이 됨

Generate Physics Shape:
- 2D 물리 충돌용 콜라이더 자동 생성
- UI면 보통 불필요
```

3. Advanced 설정:
```plaintext
sRGB (Color Texture):
- 색상 텍스처의 감마 보정 여부
- UI 텍스처는 보통 체크

Alpha Source:
- None: 알파 채널 무시
- Input Texture Alpha: 원본 이미지의 알파 채널 사용
- From Gray Scale: 회색조 값을 알파로 변환

Alpha Is Transparency:
- 알파 채널을 투명도로 사용할지 여부
- 투명도가 필요한 UI는 체크

Read/Write:
- 런타임에 텍스처 데이터 수정 가능 여부
- 성능에 영향을 주므로 필요한 경우만 체크

Generate Mipmaps:
- 작은 버전의 텍스처 자동 생성
- UI는 보통 불필요
```

![[Pasted image 20241028175510.png]]

4. 텍스처 렌더링 설정:
```plaintext
Wrap Mode:
- Repeat: 텍스처를 반복해서 표시
- Clamp: 텍스처 가장자리 픽셀을 늘려서 표시
- Mirror: 텍스처를 뒤집어가며 반복
- Mirror Once: 한번만 뒤집어서 반복
(UI의 경우 대부분 Clamp 사용)

Filter Mode:
- Point: 픽셀 그대로 표시 (픽셀아트에 적합)
- Bilinear: 부드러운 보간 (일반적인 UI에 적합)
- Trilinear: Mipmap 레벨 간 부드러운 전환
(선명한 UI는 Point, 일반 UI는 Bilinear 권장)

Aniso Level:
- 비스듬한 각도에서 텍스처 선명도 향상
- 0~16 값 (높을수록 선명하지만 성능 영향)
- UI는 보통 1이면 충분
```

5. Platform 별 설정:
```plaintext
Max Size:
- 텍스처의 최대 해상도 제한
- 원본보다 큰 경우 영향 없음
- 메모리 관리를 위해 적절히 설정

Resize Algorithm:
- Mitchell: 선명도 유지하며 크기 조절 (기본값)
- Bilinear: 부드러운 크기 조절
- Lanczos: 날카로운 엣지 보존 (텍스트에 적합)

Format:
- RGBA 32 bit: 최고 품질, 큰 용량
- RGBA 16 bit: 중간 품질, 중간 용량
- RGB 24 bit: 알파 없는 고품질
- 기타 압축 포맷들 (플랫폼 별로 다름)
(UI는 보통 RGBA 32 bit 권장)

Compression:
- None: 무압축 (최고 품질, 큰 용량)
- Normal Quality: 일반적인 압축
- High Quality: 고품질 압축
- Low Quality: 저품질 압축
(UI는 보통 None 또는 High Quality 권장)

Use Crunch Compression:
- 특수 압축 알고리즘 사용
- 로딩 시간은 늘지만 용량 크게 감소
- UI에는 보통 불필요
```