- 요약) React는 최초 로딩 시, 사이트에서 사용되는 **모든** 라이브러리를 한꺼번에 로딩한다. 이 과정에서 초기 로딩 속도가 느려지는 이슈가 발생할 수 있음.

## 대처법 
- ChatGPT님과의 질답 요약

1. **'특정 단일' 컴포넌트에서만 사용되는 라이브러리는 `React.lazy` 기능을 사용하는 게 더 적합합니다.** 이 방법을 통해 해당 컴포넌트가 실제로 필요할 때만 관련 라이브러리나 모듈을 로드하게 되므로, 초기 로딩 시간을 단축시키고 리소스 사용을 최적화할 수 있습니다. 동적 임포트(`import()`)를 사용하여 컴포넌트를 로드함으로써, 필요할 때만 해당 컴포넌트와 그 의존성들을 가져오게 됩니다.

```tsx
import React, { Suspense } from 'react';

// React.lazy를 사용하여 동적으로 불러올 컴포넌트 정의
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <LazyComponent />
      </Suspense>
    </div>
  );
}
```
> `Suspense`는 동적 임포트 중 발생하는 로딩 시간을 처리할 대체 컴포넌트로 생각하면 됨

-  또, `App.tsx`의 라우팅 기능에 대해서도 적용이 가능하다 -> 즉 최초에 모든 페이지를 로딩시키는 게 아니라, 필요할 때마다 해당 페이지들을 로딩시키는 것
```tsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';

const Home = lazy(() => import('./Home'));
const About = lazy(() => import('./About'));

function App() {
  return (
    <Router>
      <Suspense fallback={<div>Loading...</div>}>
        <Switch>
          <Route exact path="/" component={Home}/>
          <Route path="/about" component={About}/>
        </Switch>
      </Suspense>
    </Router>
  );
}
```

    
2. **'복수 개의 여러' 컴포넌트에서 사용되는 공통 기능 또는 라이브러리는 `manualChunk` 기능을 사용하는 게 더 적합합니다.** 여러 페이지나 컴포넌트에서 공통으로 사용되는 라이브러리들을 별도의 청크로 분리함으로써, 해당 청크를 필요로 하는 첫 번째 접근 시에만 로드되고, 이후에는 브라우저 캐시를 통해 재사용됩니다. 이는 공통 리소스의 로딩을 최적화하고, 전체적인 애플리케이션의 성능을 향상시키는 데 도움이 됩니다.