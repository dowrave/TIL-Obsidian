- 기본적인 Quill의 매커니즘으로 보인다. 
```tsx
  const handleImageInsert = () => {
  
	// 어떤 파일을 넣을지 나오는 상자
    const input = document.createElement('input');
    input.setAttribute('type', 'file');
    input.setAttribute('accept', 'image/*');
    input.click();

	// 여기부턴 아래에 설명
    input.onchange = (event) => {
      const file = event.target.files[0];

      if (file) {
        const reader = new FileReader();

        reader.onload = (e) => {
          const dataUrl = e.target.result;

          // 이미지를 에디터에 추가
          const range = quillRef.current.getEditor().getSelection(true);
          quillRef.current.getEditor().insertEmbed(range.index, 'image', dataUrl, 'user');
        };

        // 파일을 Base64로 변환
        reader.readAsDataURL(file);
      }
    };
  };
const modules = {
    toolbar: {
      handlers: {
        'image': handleImageInsert,  // 이미지 삽입 핸들러
      },
    },
  };
```
1. `input.onchange` : 파일을 선택한 후 발생하는 이벤트를 처리한다. 
2. `FileReader` : 파일의 내용을 비동기적으로 읽어오는 역할.
3. `reader.onload` : 파일 읽기가 성공적으로 완료될 때 실행됨. 그 값이 `dataUrl`에 저장됨.
4. `reader.readAsDataURL(file)` : 파일의 내용을 `base64`로 변환한다.