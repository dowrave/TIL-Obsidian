[출처](https://iyk2h.tistory.com/143)
### 1. Successive Halving Algorithm(SHA)
- 제한된 시간, 최소의 Loss를 갖는 하이퍼 파라미터 구성 찾기

#### 구성
1. 총 탐색에 소요되는 Budget($B$) 설정
2. $n$개의 하이퍼파라미터 설정을 랜덤하게 뽑음 $S_k$
3. $S_0$의 모델들에 동일한 $B$를 할당 $r_k = [\frac{B}{|S_k||log_2(n)}]$
4. 학습 Loss 및 중간 Loss 추출, 중간 Loss를 기준으로 성능이 좋지 않은 하이퍼 파라미터 설정을 절반씩 버린다 $S_k -> S_{k+1}$ 
5. 1개의 하이퍼파라미터 설정이 남을 때까지 반복함

##### 단점 
- $B/n$에 따른 `Exploration`, `Exploitation`의 비율이 정해지므로, B, n이라는 별도의 하이퍼파라미터가 또 생김

##### 의문점
- 하이퍼파라미터 쌍은 랜덤하게 고르는 거 아님? 랜덤 서치를 조금 더 빠르게 하는 방법으로만 보임 -> 근데 `Exploration`, `Exploitation`이라는 내용이 나오는 걸 봐서는 하이퍼 파라미터도 그냥 고르는 건 아닐 것 같음.

### 2. HyperBand
- SHA의 단점인 B, n 설정을 정해주는 알고리즘

#### 구성
1. $R$ : 하이퍼 파라미터 1개의 설정에 최대로 할당할 budget 설정
2. $\eta$ : SHA의 매 step마다 줄어드는 하이퍼파라미터 쌍 개수(= 늘어나는 budget 비율) 설정 
	- (SHA에서는 2였음)
	- HyperBand를 제시한 사람들은 3이나 4에서 최고 성능을 얻었다고 함
3. $R$, $\eta$에 따라 SHA를 반복할 개수 및 각 SHA의 처음 스텝에서 초기화하는 설정 수와 할당되는 Budget이 결정됨
$$ s_{max} = log_{\eta}(R), \hspace{0.5cm} B = (s_{max} + 1)R $$

4. $R$을 통해 각 SHA에 들어갈 $B$, $n$(하이퍼파라미터 쌍 수)을 아래 공식으로 정해줌
$$ n = [\frac{B}{R} \frac{\eta^s}{(s+1)}], \hspace{0.5cm} r = R\eta^{-s}$$5. 각 bracket의 SHA 모두 실행

#### 예시
- $s$ 값이 `bracket`이다.  $R = 81, \eta = 3$
![[Pasted image 20221122214741.png]]
![[Pasted image 20221122215440.png]]
- $n_i, r_i$ 값은 위와 같이 정해진다.
- 즉 예를 들어 `bracket = 4`라면, 
	- 81개의 하이퍼파라미터 쌍을 1 에포크씩
	- 27개의 하이퍼파라미터 쌍을 3 에포크씩
	- ....
	- 요렇게 수행함

#### 특징
- $R$ 하나 만으로 다양한 `Exploration, Exploitation` 비율을 반영한 검색을 할 수 있다
	- 각 `Bracket`에서 $n_i, r_i$ 값이 다른 것에 주목
- 각 `Bracket`은 병렬적으로 수행할 수 있다.
- `Budget` 은 학습 시간, 데이터셋 개수, feature 등 제한될 수 있는 다양한 것들이 올 수 있음