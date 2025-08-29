---
layout: post
title: kubernetes 환경 구축
tags: [kubernetes, kubernetes 설치, kubernetes 구축]
categories: server
---



- 구성
> 1. OS : Rocky Linux9
> 2. shh


- 사전조건
> 1. Rocky Linux 설치
> 2. VM을 이용하여 Rocky Linux 인스턴스 3개 설치(마스터노드 한개, 워터노드 두개)
> 2. ssh 설치 하여 외부접속 설정


설명
 - 애플리케이션을 자동배포, 확장 및 관리를 해주는 오픈소스 플랫폼이다.
 - 여러개의 컨데이너를 쉽게 생성하고 관리하는 목적으로 사용한다.
 - 네트워크 플러그인 Calico를 사용하겠다.
 - VM을 이용하여 인스턴스 생성방법은 생략하겠다.

쿠버네티스 클러스터 구조 설명
  - 클러스터 구조는 마스터노드, 워커노드로 구분되어 있다.
  - 개발자는 마스터노트와 통신, 사용자는 워커노드와 통신하는 경우가 많다.
  - CNI(Container Network Interface) 마스터노드와 워커노드간의 유기적통신.
  - CNI를 사용하기 위해 쿠버네티스 네트워크 플러그인을 제공한다(Flannel, Calico)

쿠버네티스 클러스터 구조
  - ![alt text](kubernetes_cluster.png)


작성중 .....................................