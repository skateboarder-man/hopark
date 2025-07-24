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
> 4. .env 파일 생성
> 5. flask 웹 소스 Dockerfile 생성



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

- flask 웹 소스 Dockerfile 생성
1. 폴더 이동
```
cd /home/[사용자폴더]/flask_project/[프로젝트명]/flask/
```
2. 편집기로 Dockerfile  만들기
```
vi Dockerfile
```
3. 편집기 내용 입력
```
FROM python:latest # 파이썬 최신버전
WORKDIR /usr/src/app  # container에서 작업 위치
COPY . . # hostOS의 flask 파일을 container 작업위치에 복사
RUN python -m pip install --upgrade pip  #pip 업데이트
WORKDIR ./app # container의 작업 위치 변경
RUN pip install -r requirements.txt #pip를 이용하여 flask 웹 프로젝트시 필요한 라이브러리 설치 목록 및 버젼 목록을 이용하여 다운로드
CMD gunicorn --bind 0.0.0.0:8001 app:app  # flask app.py 실행, 0.0.0.0 모든 IP 접속 허용 8001 포트 허용
EXPOSE 8001
```

- nginx(web sever) conf 파일 만들기
1. 폴더 이동 및 폴더만들기
```
cd /home/[사용자폴더]/flask_project/[프로젝트명]/
mkdir nginx
cd ./nginx
```
2. nginx 설정 파일 만들기
```
vi default.conf
```
3. 편집기 내용 입력
```
upstream myweb{
    server flask:8001;  # docker-compose.yml의 services 명과 같아야 한다.
}
server {
    listen       81;
    server_name localhost;
    location / {
        proxy_pass http://myweb;
    }
}
```

- nginx Dockerfile 만들기
1. 폴더 이동
```
cd /home/[사용자폴더]/flask_project/[프로젝트명]/nginx
```
2. Dockerfile 만들기
```
vi Dockerfile
```
3. 편집기 내용입력
```
FROM nginx:alpine
RUN rm /etc/nginx/conf.d/default.conf ##nginx 설정파일을 container의 /etc/nginx/conf.d/default.conf 삭제
COPY default.conf /etc/nginx/conf.d/ #nginx 설정파일을 container의 /etc/nginx/conf.d/default.conf 이동
CMD ["nginx", "-g", "daemon off;"]
```


- docker-composes.yml 만들기
1. 폴더 이동
```
cd /home/[사용자폴더]/flask_project/[프로젝트명]
```
2. 편집기 열기
```
vi docker-composes.yml
```
3. 편집기 입력 

```
version: '3.8'
services:
  nginx:
    build: ./nginx
    networks:
      - flasknet
    ports:
      - "81:81"
    depends_on:
      - flask
    restart: always
  
  flask:
    build: ./flask
    networks:
      - flasknet
    container_name: flask
    ports:
      - "8001:8001"
    restart: always

networks:
  flasknet:
```

- docker compose 실행 하여 container 등록
1. compose 실행
```
docker compose up -d --build
```

2. container 등록 확인
```
docker container ls
```

- 외부 접속 가능하도록 포트열어주면 외부에서 접속이 가능하다.
![alt text](flask_strucher.png)


