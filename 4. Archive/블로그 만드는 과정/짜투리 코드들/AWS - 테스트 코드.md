
- `Django - settings.py`
```python
# 업로드 테스트
try:
    response = s3_client.put_object(
        Bucket=AWS_STORAGE_BUCKET_NAME,
        Key='test_object.txt',
        Body=b'Test content',
    )
    print(f"Test object uploaded successfully. ETag: {response['ETag']}")
except boto3.exceptions.ClientError as e:
    if e.response['Error']['Code'] == 'AccessDenied':
        print("Access denied. Check the IAM role and bucket policy.")
    elif e.response['Error']['Code'] == 'SignatureDoesNotMatch':
        print("Invalid AWS access key or secret key.")
    else:
        print(f"Error uploading object: {e}")
```
> 스토리지 버킷에 잘 업로드됨;