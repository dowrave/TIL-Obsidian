- 로컬 스토리지에 토큰을 저장했는데도 새로고침을 하면 로그인이 풀리는 이슈가 있음
- 이에 대한 해결법
- (React, Redux) 사용

---
### 1. 애플리케이션 또는 컴포넌트 마운트 시 로그인 상태 복원

애플리케이션의 최상위 컴포넌트(예: `App.js`)에서 로그인 상태를 확인하고, 로컬 스토리지에 저장된 토큰을 기반으로 로그인 상태를 복원합니다.

```js
// App.js 또는 다른 최상위 컴포넌트

import React, { useEffect } from 'react';
import { useDispatch } from 'react-redux';
import { loginUser } from './actions/authActions';

const App = () => {
    const dispatch = useDispatch();

    useEffect(() => {
		const username = localStorage.getItem('username);
        const token = localStorage.getItem('userToken');
        if (username && token) {
            // 여기서 백엔드에 토큰 유효성 검증 요청을 보내고,
            // 응답으로 사용자 정보를 받는다면 더 안전합니다.
            // 예를 들어, 사용자 닉네임이나 다른 신원 정보를 받을 수 있습니다.
            
            // 백엔드 요청 없이 단순히 토큰으로 상태 복원
            dispatch(loginUser('사용자닉네임', token));
        }
    }, [dispatch]);

    return (
        // 나머지 컴포넌트 렌더링
    );
};

export default App;
```
이 코드는 애플리케이션 로드 시 로컬 스토리지에서 토큰을 가져오고, 해당 토큰이 존재한다면 로그인 상태를 Redux 스토어에 반영합니다.

### 2. 보안 고려 사항

- **토큰 검증**: 보안을 강화하기 위해, 로컬 스토리지에서 토큰을 가져온 후 백엔드 서버에 해당 토큰의 유효성을 검증하는 요청을 보내는 것이 좋습니다.
- **민감한 정보 관리**: 로컬 스토리지에 사용자의 민감한 정보(예: 사용자 이름, 이메일 등)를 저장하는 것은 보안상 좋지 않습니다. 가능하다면 토큰만 저장하고, 사용자 정보는 서버에서 검증 후 제공받는 것이 바람직합니다.

### 3. 상태 관리

- 이 접근 방식은 Redux를 통해 애플리케이션의 상태를 중앙에서 관리하며, 새로고침 시에도 사용자의 로그인 상태를 유지할 수 있도록 합니다.
- 사용자가 로그아웃을 할 때는 로컬 스토리지에서 토큰을 제거하고, Redux 스토어의 상태를 초기화해야 합니다.