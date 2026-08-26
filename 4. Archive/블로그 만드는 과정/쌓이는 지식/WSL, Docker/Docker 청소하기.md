- 빌드와 이미지 정리를 반복하다보면 쌓이는 것들이 있다
- 그것들을 제거하고 드라이브의 숨통을 틔여주자

1. **빌드 캐시 제거**
	`docker builder prune`

5. **단일 노드에서 모든 리소스 제거**
	`docker system prune -a --volumes`

6. **캐시된 리졸브 파일 제거**
	`rm -rf /etc/docker/registry.json`