```python
df.filter(regex = 'col')
```
- 열 이름에 `col`이 들어간 열들을 보여줌

```python
list(df.filter(regex = 'col'))
```
- `col`이라는 문자열이 들어간 열 이름들을 리스트로 나타냄 
- 따라서 많은 열들을 사용하는 상황에 쓸 수 있음
