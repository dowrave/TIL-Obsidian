```python
from PIL import Image, ImageDraw

def create_gradient_image(width, height, gradient_width):
    # 새 이미지 생성
    image = Image.new('RGBA', (width, height), (0, 0, 0, 0))
    draw = ImageDraw.Draw(image)

    # 왼쪽 1/4에 검은색 채우기
    left_quarter = width // 4
    draw.rectangle([0, 0, left_quarter, height], fill=(0, 0, 0, 255))

    # 그라데이션 생성
    for x in range(gradient_width):
        alpha = int(255 * (1 - x / gradient_width))
        draw.line([(left_quarter + x, 0), (left_quarter + x, height)], fill=(0, 0, 0, alpha))

    return image

# 이미지 크기 설정 (예: 1920x1080)
width, height = 1920, 1080
gradient_width = width // 4  # 그라데이션 너비를 이미지 너비의 1/4로 설정

# 이미지 생성
gradient_image = create_gradient_image(width, height, gradient_width)

# 이미지 저장
gradient_image.save('gradient_panel.png')

print("이미지가 생성되었습니다: gradient_panel.png")
```