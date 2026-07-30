- 파이썬은 비교적 설치가 편했는데, 타입스크립트는  약간의 사전 준비가 필요하다.
- 윈도우 기준, VS Code 사용

1. 프로젝트 폴더를 만듦
2. 아래 명령어들을 입력
```sh
npm init
```
- 뭐가 막 나오는데 그냥 엔터 난사했음. 연습용이니까 막 엔터 쳐도 무방해보임
- 이러면 package.json이 생긴다

- 타입스크립트 설치
```sh
npm install typescript --save-dev
npm install -g ts-node
```

```sh
npx tsc --init # ts 컴파일러 초기화
```

3. 이후로는 터미널(`ctrl + j`)을 켠 다음, `ts-node {스크립트 이름}`으로 실행해서 테스트해보면 됨
```sh
ts-node {파일명.ts}
```

