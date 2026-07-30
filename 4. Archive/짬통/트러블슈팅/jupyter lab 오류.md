### 문제 
- `anaconda prompt`에서 `jupyter lab` 실행 시 `win32api.dll`을 찾을 수 없다는 식의 오류가 뜸

### 해결법
1. `pip install pywin32==225` 설치
2. `Scripts` 경로에 갔을 때 `pywin32_postinstall.py` 파일이 있는지 확인
	- 만약 있다면 해당 경로에서 `pyhton pywin32_postinstall.py -install` 입력
3. `jupyter lab` 다시 실행해보셈

#### 과정
- `conda install -c anaconda pywin32` <- 이거는 나한테 도움이 되지 않았음
	- 스크립트만 하루 종일 돌렸는데 오류 뜨고 끝남 ㅡㅡ