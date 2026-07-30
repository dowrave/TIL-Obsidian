

- 분할 정복을 이용하는 방법. [[피보나치 수열]]
$$
\left( \begin{matrix} 1 & 1 \\ 1 & 0 \\ \end{matrix}\right)^n = \left( \begin{matrix} F_{n+1} & F_{n} \\ F_{n} & F_{n-1}\end{matrix}\right)
$$
- 시간복잡도는 $O(log(N))$  `백준 11444번 피보나치수 6`
```python

base_matrix = [[1, 1], [1, 0]]
dct = {}
dct[0], dct[1] = 0, 1 # 0번째 피보나치 수는 0, 1번째 피보나치 수는 1이다
X = 1000000007

def matmul(A, B):
    new_matrix = [[0] * 2 for _ in range(2)]
    for i in range(2):
        for j in range(2):
            for k in range(2):
                new_matrix[i][j] += (A[i][k] * B[k][j]) % X
                new_matrix[i][j] = new_matrix[i][j] % X
    return new_matrix

def matrix_fibo(n):
    
    if n == 1: # 가장 아래 지점은 0인데 n번째 피보나치 행렬의 매트릭스는 n=1부터여야 말이 됨
        return base_matrix
    
    if n not in dct.keys():
        temp = matrix_fibo(n // 2)
        temp = matmul(temp, temp)
        
        if n %2 == 0:
            dct[n] = temp[0][1]
            return temp
        
        else:
            temp = matmul(temp, base_matrix)
            dct[n] = temp[0][1]
            return temp

    # 값이 있다면 그냥 value만 리턴
    return dct[n]
    
```