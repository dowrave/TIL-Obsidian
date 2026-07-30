- Quill 에디터는 이미지를 `base64`로 저장하고 있다.
```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..." alt="image">
```

- 따라서 이를 백엔드에 파일로 저장하기 위해서는, Blob 객체로 변환할 필요가 있다.
	- Blob는 파일처럼 간주, 처리하기 편리함
```tsx
const dataURItoBlob = (dataURI) => {
  // 1. 데이터와 MIME 타입 부분을 분리
  const byteString = atob(dataURI.split(',')[1]);
  const mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];

  // 2. base64로 인코딩된 데이터를 디코딩
  const ab = new ArrayBuffer(byteString.length);
  const ia = new Uint8Array(ab);

  for (let i = 0; i < byteString.length; i++) {
    ia[i] = byteString.charCodeAt(i);
  }

  // 3. ArrayBuffer를 이용하여 Blob 객체 생성
  return new Blob([ab], { type: mimeString });
};
```
> 1. `atob` : Base64로 인코딩된 문자열을 디코딩하는 함수이다. 즉, ASCII -> 이진 데이터로 변환
> 2.  `Base64` : 이진 데이터를 ASCII 문자로 변환하는 인코딩 방식 중 하나이다.
> 3. `MIME` : 타입으로, 해당 데이터가 **어떤 종류의 데이터**인지를 나타내는 문자열이다. 
> 	- 위에서 `src` 부분을 보면 됨 : `data:image/png;base64,`에서 `image/png` 부분이다.
> 4. `ArrayBuffer` : 고정 크기의 이진 데이터 버퍼로, 전달되는 숫자는 바이트 수이다.
> 5. `Uint8Array` : `Uint8`을 나타낸다. `ab` 배열을 기반으로 하며, 일부 또는 전체를 사용한다. 현재는 각 원소에 값이 없는 상태.

- 그 결과 `ia`는 `byteString`의 길이를 갖는 0 ~ 255을 갖는 배열이 된다.
---
- `chatGPT`가 들어준 예시
```
const byteString = "\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR\x00\x00\x01\x00\x00...";
const ia = new Uint8Array(ab);
[137, 80, 78, 71, 13, 10, 26, 10, 0, 0, 0, 13, 73, 72, 68, 82, ...]
```
---
> 6. 반복문 부분 : 각 문자를 16진수로 변환, `ia[i]`에 할당한다.
> 7. `Blob` : 이진 데이터의 덩어리를 나타내는 JS 객체로, `Binary Large Object`의 줄임말이다. 대용량의 이진데이터를 캡슐화하고 처리하기 위한 용도로 사용한다.
> 	- 일반적으로 **파일**과 유사한 형태로 간주된다.
> 	- 여러 형태의 데이터를 담을 수 있으며, 주로 **이미지, 오디오, 비디오, 텍스트** 등의 다양한 이진 데이터를 표현하는 데 사용된다.