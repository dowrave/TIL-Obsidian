```tsx
const codeBlockCopy = () => {
        // 코드 블럭 우측 상단에 복사-붙여넣기 구현
        const codeBlockElements = document.querySelectorAll('pre.ql-syntax > code')
        codeBlockElements.forEach(codeBlockElement => {
            const copyButton = document.createElement('button');
            const language = codeBlockElement.className.split(" ").pop()?.slice(9); // language- 를 제거
            copyButton.textContent = language
            copyButton.classList.add('copy-button');
        
            codeBlockElement.parentNode.appendChild(copyButton);
        
            // const clipboard = new ClipboardJS(".copy-button");
            const clipboard = new ClipboardJS(copyButton, {
                text: function() {
                    return codeBlockElement.textContent;
                }
            });
        
            const messageElement = document.createElement("span");
            messageElement.textContent = "복사되었습니다!"
            messageElement.classList.add('message-element');

            codeBlockElement.parentNode.appendChild(messageElement);

            clipboard.on("success", (e) => { // e는 이벤트 객체
                // 성공 시 실행
                // messageElement.style.display = 'none';
                messageElement.classList.add('active')
                e.clearSelection(); // 드래그 영역 해제
        
                // 뒤의 {숫자}ms 후 내부 함수 실행
                setTimeout(() => {
                    console.log("Timer executed!");
                    messageElement.classList.remove('active')
                }, 2000)
            })
        })
    }
```