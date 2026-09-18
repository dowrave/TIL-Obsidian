- 현재 어떤 작업 중인지 기록 중
# 참고
- **옵시디언으로 봐야 제대로 보인다.**
	- **옵시디언으로 작성된 만큼 깃허브의 마크다운에서는 지원하지 않는 기능들이 있을 수 있다.** `[[]]`, 이미지 첨부 방식 등이 대표적.
- `[[]]` 링크는 `유니티/보관함`이나 `작업 일지/직접 작성/일지`에 대부분 있다.

# 작업 일지
## 짭명방 
- 지난 내역 : [짭명방 프로젝트 일지 링크](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EC%9C%A0%EB%8B%88%ED%8B%B0%20-%20%EC%9E%91%EC%9D%80%20%EB%AA%85%EB%B0%A9%20%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/%EC%9E%91%EC%97%85%20%EC%9D%BC%EC%A7%80/%EC%A7%81%EC%A0%91%20%EC%9E%91%EC%84%B1)
- [[기타 참고 사항]]
- [[짭명방 - 이슈 정리]]

>[!note]
>- 할 일
>1. `TitleScene` 완성하기
>	- 게임 시작 시 연출 : 스테이지 진입할 때 도형이 멀어지는 연출이랑 양 끝에서 로딩 게이지가 차오르는 그 2가지까지는 구현하고 싶다.
>	- 타이틀 씬에 쓸 음악도 필요함
>- 엔딩 크레딧 관련
## 최근 기록
- [[짭명방_260918 - 타이틀 씬 도형 마무리]]
>[!done]
>- TitleScene 아트 구현
>	- 엣지 굵기 변경 가능하게 변경 : `MeshTopology.Lines`에서 리본 메쉬(`MeshTopology.Triangles`)
>		- 삼각형의 도는 방향에 대해 정리
>	- VertexPulse 수정 : 동심원 -> 일정 주기로 크게 깜빡이는 패턴
>		- 각도에 따른 페이딩 계산은 스크립트에서 셰이더 그래프로 옮김
>		- 어두운 버텍스에는 VertexPulse가 나타나지 않음


## 짭명방
- [짭명방 프로젝트 일지 깃허브 링크(프로젝트 자체는 Private 전환)](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EC%9C%A0%EB%8B%88%ED%8B%B0%20-%20%EC%9E%91%EC%9D%80%20%EB%AA%85%EB%B0%A9%20%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/%EC%9E%91%EC%97%85%20%EC%9D%BC%EC%A7%80/%EC%A7%81%EC%A0%91%20%EC%9E%91%EC%84%B1)
## 블로그
- [React + Django 프로젝트 일지 월별 작업 기록 깃허브 링크](https://github.com/dowrave/TIL/tree/main/Obsidian/1.%20Projects%2C%20Ongoing/%EB%B8%94%EB%A1%9C%EA%B7%B8%20%EB%A7%8C%EB%93%A4%EA%B8%B0/%EC%9B%94%EB%B3%84%20%EC%9E%91%EC%97%85%20%EA%B8%B0%EB%A1%9D)
