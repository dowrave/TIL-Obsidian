- 당연히 일반적으로는 가장 마지막에 쓰이는 `index.css`에 커스텀 클래스 이름을 만들고, 그 값을 넣으면 된다.
- 하지만 여기서는, `h-9/10` 같이, 일반적인 `css` 문법으로 생성할 수 없는 클래스 이름을 넣는 방법을 설명한다.

---

## 구현
- `tailwind.config.js`에 아래처럼 항목을 추가한다.
```tsx
module.exports = {
	content: [
	],
	// 여기에 별도의 css 설정을 추가한다
	theme: {
		extend: {
			height: {
				'9/10' : '90%' // h-9/10 = 높이 90%
			}
		}
	}
}
```

- `tailwind css`는 `h-숫자`나 `h-분수` 등을 사용하는데, 숫자는 `rem` 개념이고 분수는 `%` 개념이다.  
- 그러나 `90%`를 나타내는 개념은 사전에 정의되어 있지 않아서, 이를 tailwind CSS의 문법을 따르면서 정의해줄 필요가 있을 때 위 방법을 쓸 수 있다.

### 참고
- 아예 완전히 새로운 클래스를 추가하는 방법도 설명해둔다.
```tsx
const plugin = require('tailwindcss/plugin');

module.exports = {
  plugins: [
    plugin(function({ addUtilities, theme, e }) {
      const newUtilities = {
        '.custom-utility': {
          backgroundColor: theme('colors.red.500'), // theme 함수를 사용하여 테마 값 접근
          color: 'white',
          padding: '1rem',
        },
      };
      addUtilities(newUtilities, ['responsive', 'hover']);
    }),
  ],
}
```

- 생각보다 복잡하기 때문에, 일반적으로 `React` 프로젝트에서는 `index.css`를 이용하는 편이 훨씬 직관적이고 간편해보인다.

- 위의 코드는 `index.css`에서 아래처럼 정의할 수 있기 때문이다.
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  .custom-utility {
    @apply bg-red-500 text-white p-4;
  }
}
```
