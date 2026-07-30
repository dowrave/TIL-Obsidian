- Quill의 커스텀 블랏을 이용하려다 그냥 기존에 있는 헤더를 이용하기로 합니다.

```tsx
const toggleContentVisible = (e: React.MouseEvent<HTMLElement>) => {
    const targetTagName = e.target.nodeName
    if (e.target && ['H1', 'H2', 'H3'].includes(targetTagName)) {
        let nextElement= e.target.nextElementSibling;

        // 접는 범위 지정
        const nextHeaders = ['H1']; // 재할당이 아니라면 const 선언 가능
        if (targetTagName === 'H2') {nextHeaders.push('H2')}
        if (targetTagName === 'H3') {nextHeaders.push('H2', 'H3')}

        while (nextElement && !nextHeaders.includes(nextElement.nodeName)) {
            nextElement.style.display = nextElement.style.display === 'none' ? '' : 'none'
            nextElement = nextElement.nextElementSibling;
        }
    }
}

export default toggleContentVisible;
```
> 요는 각 헤더(h1, h2, h3)에 대해 다음 자기와 비슷한 태그가 올 때까지의 영역의 `display`를 `none`으로 수정하는 것이다.


