- 스포일러 경고 문구 추가 과정

### 1. 커스텀 블랏 작성
```tsx
import { Quill } from 'react-quill';
import 'react-quill/dist/quill.snow.css';

// 스포일러 경고 요소를 위한 사용자 정의 블럿 포맷 정의
const SpoilerWarningBlot = Quill.import('blots/block');

export default class SpoilerWarning extends SpoilerWarningBlot {
  static create() {
    const node = super.create();
    node.setAttribute('class', 'spoiler-warning');
    // node.setAttribute('style', 'width: 100%; height: 100px; border: 2px solid red;'); // 빨간 테두리선 추가
    node.setAttribute('class', 'w-full h-20 border border-red-400 bg-red-50 my-1 p-2 rounded-md')
    // node.setAttribute('contenteditable', false)

    const title = document.createElement('strong');
    title.setAttribute('class', 'text-lg xl:text-xl align-center');
    title.textContent = '스포일러 주의!';

    const message = document.createElement('span');
    message.setAttribute('class', 'text-sm xl:text-base align-center xl:mt-1')
    message.textContent = '아래의 내용은 스포일러를 포함하고 있습니다.';

    node.appendChild(title);
    node.appendChild(message);

    return node;
  }

  constructor(scroll: typeof Quill, domNode: HTMLElement) {
    super(scroll, domNode);
    this.domNode.setAttribute('contenteditable', 'false');
  }

  static formats() {
    return true;
  }
}

SpoilerWarning.blotName = 'spoiler-warning';
SpoilerWarning.tagName = 'div';
```
> 이 때 `formats`, `toolbar`에는 `spoiler-warning`으로 구현하면 됨

### 2. QuillEditor에 삽입
- 외부에 별도의 컴포넌트로 구현했다면 크게 아래 4가지로 넣으면 된다.
	1. `register`
	2. `toolbar`
	3. `handler`
	4. `formats`

```tsx
// 등록
	// 모듈 : 에디터의 기능 확장 (toolbar, clipboard, history, syntax 등등)
	// 포맷 : 에디터의 텍스트, 콘텐츠 스타일 기능 정의 (bol,d italic, link, image, video 등등)
	// 일반적인 등록 : Quill.register('blots/block', CustomBlot 같이)
Quill.register(SpoilerWarning)

const QuillEditor = ({ content, onChange, backend, quillRef, test=false }: {
	
    const spoilerAlertHandler = useCallback((quill: ReactQuill) => {
        const blotLength = 33
        const range = quill.getEditor().getSelection(true); // 현재 커서 위치
        if (range) {
            const index = range.index;
            quill.getEditor().insertEmbed(range.index, 'spoiler-warning', true, 'user');
            quill.getEditor().setSelection(index + blotLength, 0);   
        }
    }, [])
    
    const modules = {
            toolbar: {
                container : [
                    ['spoiler-warning'],
                    ],
                'spoiler-warning': () => spoilerAlertHandler(quillRef.current!)
                
      }

    // // 에디터에서 허용하는 서식
    const formats = [
        'spoilerWarning', 
    ]
```
> 주의) Handler는 useCallback을 사용해서 구현하는 게 안전하다 : useCallback을 쓰지 않으면 타이핑 시 컴포넌트가 날아가는 현상이 있음(Handler 함수에 아무 것도 없더라도!)


## 그러나 이런 문제가 있다
- 글을 저장하고 수정할 때 기존에 설정한 CustomBlot이 없어진다
	- 외부 `div` 태그가 사라진다. 근데 `strong`이랑 `span` 태그는 남아있다.
