- `Windows 기능 켜기/끄기` -> `Linux용 Windows 하위 시스템 사용하기` 체크

## 1. 윈도우 10 -> 우분투

- 리눅스 파일 시스템 경로 : `c:\Users\사용자이름\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs`
	- `rootfs` 폴더가 없는데? ㅁㅊ
- 근데 탐색기 왼쪽에 아예 `Linux`가 떠 있음


## 2. 우분투 -> 윈도우 10
```sh
cd ../../mnt/c
```
- 윈도우의 c 드라이브임
- 폴더 복사는 `cp -r 원본이름 복사본이름`으로 가능
