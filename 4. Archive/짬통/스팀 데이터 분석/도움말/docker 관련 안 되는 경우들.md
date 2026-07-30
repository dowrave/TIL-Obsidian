1. `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`
```sh
sudo service docker start
# 비밀번호는 바탕화면에 O

```

2. `Error response from daemon: pull access denied for pysql, repository does not exist or may require 'docker login': denied: requested access to the resource is denied`
```sh
docker login
```
- 아이디는 쓰던거, 비밀번호는 요즘 꺼 알파벳만 

