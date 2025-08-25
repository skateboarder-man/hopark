---
layout: post
title: 파이썬의 flask를 이용하여 웹페이지 만들기
tags: [python, web, flask]
categories: python
---


목적 : 지출 기안서 자동작성 프로젝트

기능 : execle문서 출력, word문서 출력, 기본적인 CRUD 기능

설명 : 전체적인 웹페이지 제작방법을 설명하려니 내용이 너무 많다. 소스를 깃허브에 올렸으니 다운받아 사용하길 바란다. flask 웹 제작의 난이도는 어렵지 않으니 충분히 분석할 수 있을 것이라 생각한다.

![alt text](flask_structure.png)


프로젝트 구조.
- app.py: Flask 애플리케이션의 진입점입니다.
- config.py: Flask 애플리케이션의 설정을 정의하는 파일입니다.
- requirements.txt: 프로젝트에 필요한 Python 패키지들의 목록을 담은 파일입니다.
- migrations/: 데이터베이스 마이그레이션 스크립트를 저장하는 디렉토리입니다.
- venv/: 가상 환경 디렉토리입니다.
- board_app/: Flask 애플리케이션의 메인 모듈입니다.
- models/: 데이터베이스 모델을 정의하는 모듈들을 저장합니다.
 (DB 직접 구성시 필요 없음.)
- schemas/: 데이터 직렬화를 위한 Marshmallow 스키마를 정의하는 모듈들을 저장합니다.
 (DB 직접 구성시 필요 없음.)
- static/: 정적 파일들(css, js 등)을 저장하는 디렉토리입니다.
- templates/: HTML 템플릿 파일들을 저장하는 디렉토리입니다.
- views/: Flask 뷰 함수를 정의하는 모듈들을 저장합니다.
- apis/: API 엔드포인트를 정의하는 모듈들을 저장합니다.


실행 방법

1. 사전조건
 - visual studio code 설치
 - 파이썬 설치

2. 공유한 github 링크 접속하여 다운로드 받기
 - [flask 웹 소스 링크](https://github.com/skateboarder-man/expense_hopark) https://github.com/skateboarder-man/expense_hopark

3. 소스코드 열기
 - visual studio code를 이용하여 다운받은 소스코드를 file -> open folder를 이용하여 폴더 열기

4. .env 파일 만들기
 - DB 설정 정보 및 노출 되면 안되는 중요한 정보를 관리한다.
```
# 운영 DB 정보
FLASK_APP=expense_app
PRODUCTION_DB_USERNAME='사용자 명'
PRODUCTION_DB_PASSWORD='DB 비밀번호'
PRODUCTION_DB_IP='192.168.0.1'
PRODUCTION_DB_HOST='3306'
PRODUCTION_DB_NAME='데이터베이스 명'
# 테스트 DB 정보
DEVELOPMENT_DB_USERNAME='사용자 명'
DEVELOPMENT_DB_PASSWORD='DB 비밀번호'
DEVELOPMENT_DB_IP='192.168.0.1'
DEVELOPMENT_DB_HOST=3306
DEVELOPMENT_DB_NAME='테스트 데이터베이스 명'
#개발 운영 변경 처리
FLASK_ENV=development #production #development  
#난수 생성 SECRET_KEY 입력
FLASK_SECRET_KEY='test 폴더의 ranHexlify.py 실행한 16진수 문자열 입력'
```

