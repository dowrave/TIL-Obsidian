```cs
transform.position = Vector3.MoveTowards(transform.position, targetPosition, speed * Time.deltaTime);
```
1. 목적:
    - **한 위치에서 다른 위치로 일정한 속도로 이동하는 벡터를 계산**합니다.
2. 매개변수:
    - `current`: 현재 위치 (`Vector3`)
    - `target`: 목표 위치 (`Vector3`)
    - `maxDistanceDelta`: 최대 이동 거리 (`float`)
3. 반환값:
    - 새로운 위치를 나타내는 `Vector3를` 반환합니다.
4. 동작 방식:
    - 현재 위치에서 목표 위치 방향으로 `maxDistanceDelta` 만큼 이동한 위치를 **계산**합니다.
    - 만약 **현재 위치와 목표 위치 사이의 거리가 `maxDistanceDelta`보다 작다면, 목표 위치를 그대로 반환**합니다.
5. 특징:
    - 일정한 속도로 이동: `maxDistanceDelta`를고정값으로 사용하면 프레임률에 상관없이 일정한 속도로 이동합니다.
    - 목표 지점 초과 방지: 목표 지점을 지나치지 않고 정확히 도달하게 해줍니다.
    - 선형 보간: 시작점과 끝점 사이를 선형으로 보간합니다.
6. 장점:
    - 간단하고 직관적인 사용법
    - 물리 엔진을 사용하지 않고도 부드러운 이동 구현 가능
    - 목표 지점을 정확히 도달할 수 있음
7. 주의사항:
    - 장애물 회피나 복잡한 경로 탐색에는 적합하지 않습니다. 단순 직선 이동에 최적화되어 있습니다.