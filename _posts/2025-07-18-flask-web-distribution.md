---
layout: post
title: docker를 이용하여 flask web 프로젝트 배포
tags: [docker, flask, server, container, compose, nginx, gunicorn]
categories: server
---

- 구성
> 1. OS : Rocky Linux9
> 2. shh

- 사전조건
> 1. Rocky Linux 설치
> 2. ssh 설치 하여 외부접속 설정

- 구축 순서

- 시스템 업데이트
```
sudo dnf update -y
sudo dnf upgrade -y
```


- 도커 설치
1. 필요한 유틸설치
저장소를 쉽게 추가할 수 있도록 도와주는 유틸.
```
sudo dnf install -y dnf-utils
```
2. Docker 공식 저장소 추가
```
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```
3. Docker Engine 및 관련 패키지 설치
```
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
4. Docker 서비스 시작 및 자동 시작 설정
```
sudo systemctl start docker
sudo systemctl enable docker
```
5. Docker 서비스 상태 확인
```
sudo systemctl status docker
```
![alt text](docker_run.png)
6. 사용자로 Docker 사용 설정
현재 리눅스 사용자로 지정하여 설정
```
sudo usermod -aG docker $(whoami)
```
7. Docker 설치 확인
```
docker run hello-world
```
![alt text](docker_hello.png)

여기 까지 도커 설치 완료 입니다.

- flask compose yml을 만들어서 docker container에 등록
