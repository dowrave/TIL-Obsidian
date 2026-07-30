- 기본적인 구성은 `audioRef`을 이용함!
## 1. 전체적인 구성
### 1. 앨범아트 및 현재 재생 중인 곡의 이름

음악 플레이어의 현재 재생 중인 곡 정보와 앨범 아트를 보여주기 위해, React 컴포넌트 내에 적절한 HTML 구조를 추가합니다.

```jsx
<div className="flex items-center space-x-4">
    <img src={playList[currentTrackIndex].album_art} alt="Album Art" className="w-20 h-20 rounded" />
    <div>
        <h2 className="text-lg font-bold">{playList[currentTrackIndex].name}</h2>
        <p className="text-sm text-gray-500">{playList[currentTrackIndex].artist}</p>
    </div>
</div>
```

### 2. 현재 재생 중인 곡의 시간 및 진행상태 바

재생 시간과 진행 상태를 보여주는 슬라이더를 만들고, 사용자가 슬라이더를 조작하여 재생 위치를 변경할 수 있도록 합니다. HTML `input` 태그의 `type="range"`를 사용하여 슬라이더를 구현합니다.

```jsx
<div className="flex items-center space-x-2">
    <span className="text-xs">{currentTime}</span>
    <input
        type="range"
        min="0"
        max={duration}
        value={currentPosition}
        onChange={handleSliderChange}
        className="slider w-full h-2 bg-gray-300 rounded-lg appearance-none cursor-pointer"
    />
    <span className="text-xs">{formattedDuration}</span>
</div>
```

여기서 `handleSliderChange` 함수는 슬라이더를 조작할 때 호출되며, `audioRef.current.currentTime` 속성을 업데이트하여 오디오 재생 위치를 조정합니다.

### 3. 볼륨 조절 바

볼륨 조절도 슬라이더를 이용하여 구현하며, 사용자가 슬라이더를 조작할 때 오디오의 볼륨을 조절할 수 있도록 합니다.

```jsx
<div className="flex items-center space-x-2">
    <i className="fas fa-volume-down"></i>
    <input
        type="range"
        min="0"
        max="100"
        value={volume * 100}
        onChange={handleVolumeChange}
        className="slider w-full h-2 bg-gray-300 rounded-lg appearance-none cursor-pointer"
    />
    <i className="fas fa-volume-up"></i>
</div>
```

여기서 `handleVolumeChange` 함수는 볼륨 조절 슬라이더를 조작할 때 호출되며, `audioRef.current.volume` 속성을 업데이트하여 오디오의 볼륨을 조정합니다.

위 코드들을 기존 코드에 적절히 통합하여 사용할 수 있습니다. 각 기능에 필요한 state 관리 로직 (예: 현재 재생 시간, 볼륨 등)을 추가하여 전체적인 사용자 경험을 개선할 수 있습니다. Tailwind CSS 클래스를 활용하여 스타일링도 적용해주세요.

## 2. 구성에 사용되는 상태 / 함수들

재생 시간과 진행 상태 바를 제어하기 위해 필요한 React의 state 관리와 함수들을 구현하려면 몇 가지 단계가 필요합니다. `duration`, `currentPosition`, `handleSliderChange`, `formattedDuration`을 각각 구현하는 방법을 설명드리겠습니다.

### State 정의 및 초기화

먼저, React 컴포넌트의 state를 사용하여 오디오의 현재 위치, 전체 길이, 현재 볼륨 등을 저장합니다. 이는 `useState` 훅을 사용하여 구현합니다.

```javascript
import React, { useState, useEffect, useRef } from 'react';

const [currentPosition, setCurrentPosition] = useState(0);
const [duration, setDuration] = useState(0);
```

### 오디오 메타 데이터 로드

`audio` 요소의 `loadedmetadata` 이벤트를 통해 오디오의 전체 길이를 가져올 수 있으며, 이를 `duration` 상태에 저장합니다.

```javascript
const audioRef = useRef();

useEffect(() => {
    const audio = audioRef.current;
    const setAudioData = () => {
        setDuration(audio.duration);
    };

    audio.addEventListener('loadedmetadata', setAudioData);

    return () => {
        audio.removeEventListener('loadedmetadata', setAudioData);
    };
}, []);
```

### 진행 상태 바 조작

`input` 요소의 `onChange` 이벤트 핸들러로 사용되는 `handleSliderChange` 함수는 사용자가 슬라이더를 조작할 때 호출됩니다. 이 함수는 슬라이더의 값(새로운 재생 위치)을 오디오 요소에 적용하고, 현재 위치 상태를 업데이트합니다.

```javascript
const handleSliderChange = (event) => {
    const newTime = event.target.value;
    audioRef.current.currentTime = newTime;
    setCurrentPosition(newTime);
};
```

### 재생 시간 포맷팅

`formattedDuration`을 계산하는 방법은 일반적으로 초를 분:초 형식으로 변환하는 함수를 사용합니다. 이를 구현하는 간단한 함수 예는 다음과 같습니다.

```javascript
const formatTime = (time) => {
    const minutes = Math.floor(time / 60);
    const seconds = Math.floor(time % 60);
    const formattedSeconds = seconds < 10 ? `0${seconds}` : seconds;
    return `${minutes}:${formattedSeconds}`;
};

const formattedDuration = formatTime(duration);
```

### 현재 재생 시간 갱신

오디오가 재생될 때마다 `currentPosition`을 업데이트하려면 `timeupdate` 이벤트를 사용합니다.

```javascript
useEffect(() => {
    const audio = audioRef.current;
    const updatePosition = () => {
        setCurrentPosition(audio.currentTime);
    };

    audio.addEventListener('timeupdate', updatePosition);

    return () => {
        audio.removeEventListener('timeupdate', updatePosition);
    };
}, []);
```

위의 코드들을 종합하여 사용하면, 사용자가 진행 상태 바를 조작하여 오디오 재생 위치를 변경할 수 있고, 현재 재생 시간과 전체 재생 시간을 화면에 보여줄 수 있습니다.