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
> 3. 완성된 flask 웹 소스


- flask 웹 소스 구성
> 1. .env -> 중요 정보 (DB 접속정보, secret key 정보 등..) 관리
> 2. .dockerignore -> docker image build 시 빌드 되면 안되는 파일 목록 관리
> 3. .gitignore -> git push 할때 main에 업로되면 안되는 파일 목록 관리 (중요 : .env 파일은 절대 git에 올리면 안됨)
> 4. requirements.txt -> flask 웹 프로젝트시 필요한 라이브러리 설치 목록 및 버젼 관리

- 구축 순서
> 1. 시스템 업데이트
> 2. docker 설치
> 3. gitLab 에서 flask 웹 소스 가져오기(clone)


- 시스템 업데이트
```
sudo dnf update -y
sudo dnf upgrade -y
```


- docker 설치
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


- gitlab에서 flask 웹 source 가져오기
1. 폴더 생성
```
cd /home/[사용자폴더]/
mkdir -p flask_project/[프로젝트명]/flask/app
cd /home/[사용자폴더]/flask_project/[프로젝트명]/flask/app
git clone http://주소/flask.git . #프로젝트 git 주소 입력 (http://주소/flask.git) 끝에 (.)의 의미는 현재폴더에 가져온다.
```
2. 아이디, 비밀번호 입력
>![alt text](git_clone_username.png)
>![alt text](git_clone_password.png)
>![alt text](git_clone_success.png)


- .env 파일 만들기
1. vi 명령어로 .env 파일생성
```
cd /home/[사용자폴더]/flask_project/[프로젝트명]/flask/app
vi .env
```
2. .env 내용
> flask 웹 프로젝트 소스에 있는 .env파일 내용을 그대로 복사 붙여넣기 한다.
>![alt text](env.png)



- flask compose yml을 만들어서 docker container에 등록
