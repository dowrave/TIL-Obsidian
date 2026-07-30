
## 1. 로컬(드라이브)
- 경로 앞에 `file:///`을 붙인다.
```python
# 예시
local_path = "file://content/drive/MyDrive/Now/KR-BERT/krbert_tensorflow/models/char_ranked/model.ckpt-2000000"
```


## 2. GCS(클라우드, 버킷)
- 경로 앞에 `gs://`을 붙인다.
```python
gcs_path = "gs://your-bucket-name/models/char_ranked/model.ckpt-2000000"
```


