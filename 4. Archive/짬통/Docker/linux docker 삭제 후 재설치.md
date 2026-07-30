```sh
sudo apt-get purge -y docker-engine docker docker.io docker-ce docker-ce-cli 
sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce
```

- 컨테이너, 이미지, 볼륨, 사용자 파일 등 전부 삭제
```sh
sudo rm -rf /var/lib/docker /etc/docker 
sudo rm /etc/apparmor.d/docker 
sudo groupdel docker 
sudo rm -rf /var/run/docker.sock
```

- 설치는 [[MLOps 시스템 구축하기]]를 따라감
