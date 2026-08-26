
```tsx
// 상위 컴포넌트 : WritePost.tsx ------------------------------------------------
    const [title, setTitle] = useState(editData?.title || '')
    const [subsections, setSubsections] = useState([])
    const [subsection, setSubsection] = useState(editData?.subsection || '');
	const [content, setContent] = useState(editData?.content || '');
    
    const handleContentChange = (newContent) => {
        setContent(newContent)
    }
// ...
                <QuillEditor 
                content={content} 
                onChange = {handleContentChange}
                backend={backend} 
                quillRef={quillRef} />

// 하위 컴포넌트 : QuillEditor.tsx -----------------------------------------------
const QuillEditor: React.FC<QuillEditorProps> = React.memo(({ content, onChange, backend, quillRef }) => {

    const applyHighlight = () => {
		// 하이라이트를 적용하는 코드
    }

    const handleTextChange = (delta, oldDelta, source) => {
        const editor = quillRef.current.getEditor()
        if (source === 'user') { 
            const cursorPosition = editor.getSelection()?.index;
            if (cursorPosition !== null) {
                const [line, offset] = editor.getLine(cursorPosition);
            if (line.domNode.localName !== 'pre') {
                applyHighlight();
            }
            }
        }
        const htmlContent = editor.root.innerHTML;
    }
    useEffect(() => {
      hljs.configure({
          languages: ['jsx', 'tsx', 'python', 'sql', 'csharp', 'cpp', 'html', 'css']
      });

      // 하이라이트 적용
      if (quillRef.current) {
        const quill = quillRef.current.getEditor();
        quill.on('text-change', handleTextChange)
      }

    }, [])
    
    return (
        <div className='full-height-container'>
            <ReactQuill
                ref = {quillRef}
                theme = 'snow'
                modules = {modules}
                formats = {formats}
                value={ content }
                onChange = {handleTextChange}
            />
        </div>
    )
}
```

하이라이트를 적용하기 위해 `handleTextChange`라는 커스텀 핸들러를 넣었다. Quill 에디터에서 제공하는 `text-change` 이벤트로, 텍스트가 바뀔 때마다 호출되는 이벤트이다.

최초에는 이를 `onChange`에 넣어 `handleTextChange`에 넣었다. 

## 이슈
- `title, subsection` 등 상위 컴포넌트에서 정의된 상태를 넣고, 하위 컴포넌트의 상태를 변경할 때는 아무 이상 없었다.
- 그러나 **하위 컴포넌트(`content`)에 글을 먼저 넣은 다음 상위 컴포넌트를 변경했더니 하위 컴포넌트의 렌더링이 초기화되는 현상**이 있었다.


## 해결
- `handleTextChange`는 `useEffect` 문 내부에 이렇게 정의된다.
```tsx
        if (quillRef.current) {
        const quill = quillRef.current.getEditor();
        quill.on('text-change', handleTextChange)
      }
```

여기서 사용자는 `ReactQuill` 컴포넌트의 내장 이벤트 리스너를 직접 설정하는 것이다. 즉, 이 때 사용하는 `prop`은 `onChange`가 아니라 직접 설정한 이벤트 리스너가 된다. 따라서 
1. **`onChange`에 아무것도 전달하지 않아도 된다.**
2. **상위 컴포넌트의 상태 `content`를 변경하기 위한 핸들러 함수는 `handleTextChange`에서 호출되어야 한다.**

따라서 코드는 이렇게 수정된다.
```tsx
// 하위 컴포넌트
    const handleTextChange = (delta, oldDelta, source) => {
        const editor = quillRef.current.getEditor()
        if (source === 'user') { 
            const cursorPosition = editor.getSelection()?.index;
            if (cursorPosition !== null) {
                const [line, offset] = editor.getLine(cursorPosition);
            if (line.domNode.localName !== 'pre') {
                applyHighlight();
            }
            }
        }
        const htmlContent = editor.root.innerHTML;
        console.log('htmlContent : ' + htmlContent);
        onChange(htmlContent); // 추가
    }

    return (
        <div className='full-height-container'>
            <ReactQuill
                ref = {quillRef}
                theme = 'snow'
                modules = {modules}
                formats = {formats}
                value={ content }
                // onChange = {handleTextChange} // 주석처리
            />
        </div>
    )
```

- onChange 핸들러의 경우와 직접 설정한 text-change 핸들러의 차이는 아래 chatGPT님의 설명을 보자.


---
`ReactQuill`의 경우, `onChange` 이벤트 핸들러는 보통 네 가지 매개변수를 받을 수 있습니다: `content`, `delta`, `source`, 그리고 `editor`입니다. 이들은 각각 다음을 나타냅니다:

1. **content**: 현재 에디터의 내용을 HTML 문자열 형태로 나타냅니다.
2. **delta**: 변경 사항을 나타내는 Quill의 Delta 객체입니다.
3. **source**: 변경의 원인을 나타내는 문자열입니다 (예: 'user', 'api').
4. **editor**: 현재 Quill 에디터의 인스턴스입니다.

때로는 모든 매개변수가 필요하지 않을 수 있습니다. 예를 들어, 단지 에디터의 내용만 필요한 경우에는 `content`만 사용할 수 있습니다. 다른 경우에는 `delta`와 `source`를 사용하여 변경 사항의 성격을 파악하거나, `editor`를 사용하여 에디터의 현재 상태에 접근할 수 있습니다.

`text-change` 이벤트를 직접 사용할 경우, 이 이벤트는 보통 `delta`, `oldDelta`, `source` 세 가지 매개변수를 제공합니다:

1. **delta**: 최근에 발생한 변경을 나타내는 Delta 객체입니다.
2. **oldDelta**: 변경 이전의 Delta 상태입니다.
3. **source**: 변경의 원인을 나타내는 문자열입니다 (예: 'user', 'api').

이러한 차이는 `ReactQuill`의 `onChange` prop을 사용하는 경우와 `quillRef.current.getEditor().on('text-change', handler)`를 사용하는 경우 간의 차이에서 기인합니다. 전자는 `ReactQuill` 컴포넌트의 편의성을 위해 제공되는 API이며, 후자는 Quill 에디터의 내부 API를 직접 사용하는 것입니다.