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

master node, worker node 공동 작업
 - 관리자로 변경
  ```
  sudo -i
  ```
 - swap 비활성화 (swap이 비활성화 된 환경에서 최적으로 작동함.)
  ```
  vi /etc/fstab
  ```
  ![alt text](fstab.png) 
  주석처리 후 저장 종료(swap 비활성화)

- selinux 비활성화
  ```
  vi /etc/selinux/config
  ```
  ![alt text](selinux.png) disabled 처리 (쿠버네티스 구축 완료 후 다시 활성화 예정)

- hostname 변경
  ```
  hostnamectl set-hostnsme [호스트명]
  ```
  master, worker01, wordker02 변경처리 

- host 파일 변경처리
  ```
  vi /etc/hosts
  ```
  ![alt text](hosts.png)
  master, worker01, worker02 host 추가 입력
  ```
  reboot
  ```
  서버 재시작 해야 적용됨.
  ![alt text](host_change.png)

- overlay, br_netfilter 모듈 적용
  ```
  sudo modprobe br_netfilter
  sudo modprobe overlay
  ```
  현재 세션에 모듈을 로드한다.
  ```
  vi /etc/modules-load.d/k8s.conf
  ```
  편집기를 열고 
  ```
  br_netfilter
  overlay
  ```
  해당내용 추가하여 저장.

   ![alt text](ov_br_module.png)

  해당 작업 설명
   - overlay : 여러 레이어를 겹쳐 하나의 통합된 파일 시스템처럼 보이게 하는 기술.
   - br_netfilter : 브릿지된 네트워크 트래픽이 iptables 규칙을 통과하도록 허용.

  결론
    - 시스템 부팅 시 자동으로 필요한 커널 모듈을 로드하여 Kubernetes가 네트워크를 올바르게 구성하고 작동하도록 하기 위함.

- 네트워트 기능 설정
  ```
  vi /etc/sysctl.d/k8s.conf
  ```
  ```
  net.bridge.bridge-nf-call-iptables = 1 
  #브릿지된 IPv4 트래픽이 iptables 규칙을 통과하도록 허용
  net.ipv4.ip_forward = 1 
  #IPv4 패킷 포워딩을 활성화
  net.bridge.bridge-nf-call-ip6tables = 1
  #브릿지된 IPv6 트래픽이 ip6tables 규칙을 통과하도록 허용
  ```
  해당내용 추가하여 저장.
  ```
  sysctl --system
  ```
  변경사항을 즉시 활성화.
  ![alt text](sysctl.png)

- dnf 유틸리티 설치
  ```
  dnf install -y dnf-utils
  ```
- Docker 리포지토리 추가
  ```
  dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  ```
- install container runtime containered
  ```
  dnf install containerd.io -y
  ```
- containerd 설정 파일 백업 및 초기화
  ```
  mv /etc/containerd/config.toml /etc/containerd/config.toml.bkp
  # 원본설정 파일 백업파일로 변경처리
  containerd config default | sudo tee /etc/containerd/config.toml
  # 설정파일을 기본값으로 생성 
  vi /etc/containerd/config.toml
  # 설정파일 수정
  ```
  ```
  SystemdCgroup = true
  ```
  ![alt text](containerd_config.png)
  ```
  sudo systemctl restart containerd
  sudo systemctl enable containerd
  ```
- kubernetes 설치
  ```
  vi  /etc/yum.repos.d/kubernetes.repo
  ```
  ```
  [kubernetes]
  name=Kubernetes
  baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
  enabled=1
  gpgcheck=1
  gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
  exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
  ```
  입력후 저장
  ```
  dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
  systemctl start kubelet
  systemctl enable kubelet
  ```
  
mater node 작업
- kubernetes init 설정
  ```
  kubeadm init --control-plane-endpoint=master
  #hosts 파일에 master 정보를 입력하였기때문에 endpoint에 master 라고 입력하였음.
  ```
- root 계정접속 종료
  ```
  exit
  ```
- 사용 포트 허용
  ```
  firewall-cmd --permanent --add-port={6443,2379,2380,10250,10251,10252,10257,10259,179}/tcp
  firewall-cmd --permanent --add-port=4789/udp
  firewall-cmd --reload
  ```
  해당 포트 설명
  > 1. 6443/tcp: Kubernetes API 서버의 기본 포트.
      - 모든 클러스터 통신(예: kubectl 명령, 다른 컴포넌트의 요청)이 이 포트를 통해 이루어짐.
      - 워커 노드 및 관리자 PC가 마스터 노드에 접근하기 위해 반드시 열려 있어야함.
  > 2. 2379, 2380/tcp: etcd의 포트.
      - etcd는 클러스터의 모든 상태 정보와 설정 데이터를 저장하는 분산형 키-밸류 저장소.
      - 이 포트들은 etcd 클라이언트와 피어 통신에 사용됨.
  > 3. 10250/tcp: Kubelet API의 포트.
      - 워커 노드의 Kubelet이 마스터 노드의 API 서버와 통신.
      - Pod 상태를 보고하거나 컨테이너를 관리하는 데 사용.
  > 4. 10251/tcp: kube-scheduler의 포트.
      - Pod를 어느 워커 노드에 배치할지 결정하는 스케줄러 컴포넌트의 통신 포트.
  > 5. 10252/tcp: kube-controller-manager의 포트.
      - 클러스터의 상태를 관리하는 컨트롤러 매니저 컴포넌트의 통신 포트.
  > 6. 10257/tcp: kube-controller-manager의 Metrics 포트.
      - 클러스터 상태와 관련된 지표(Metrics)를 제공.
  > 7. 10259/tcp: kube-scheduler의 Metrics 포트. 
      - 스케줄러의 성능과 관련된 지표를 제공.
  > 8. 179/tcp, 4789/udp: Calico와 같은 CNI(Container Network Interface) 플러그인이 사용하는 포트.
      1. 179/tcp: BGP(Border Gateway Protocol) 통신에 사용
        - Calico는 BGP를 사용하여 클러스터 내의 라우팅 정보를 교환.
      2. 4789/udp: VXLAN 프로토콜에 사용.
        - Calico가 VXLAN 모드로 구성되었을 때, 노드 간의 Pod 트래픽을 캡슐화하여 전달.

- kubernetes 설정
  ```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

- Calico 설치
  ```
  kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
  ```




woekr node 작업
- 사용 포트 허용
  ```
  firewall-cmd --permanent --add-port={179,10250,30000-32767}/tcp
  firewall-cmd --permanent --add-port=4789/udp
  firewall-cmd --reload
  ```
  해당 포트 설명
  > 1. 179/tcp: Calico와 같은 CNI(Container Network Interface) 플러그인이 BGP(Border Gateway Protocol) 통신을 위해 사용.
      - 워커 노드는 이 포트를 통해 다른 노드와 라우팅 정보를 교환.
  > 2. 10250/tcp: Kubelet API 포트.
      - 마스터 노드가 워커 노드의 상태를 모니터링하고, Pod를 배치하며, 컨테이너를 관리하는 데 사용.
  > 3. 30000-32767/tcp: NodePort Services의 포트 범위.\
      - 이 범위의 포트는 외부에서 클러스터의 서비스에 접근하기 위해 열려 있어야 함.
      - Kubernetes는 이 포트를 통해 외부 트래픽을 Pod로 라우팅함.
  > 4. 4789/udp: Calico가 VXLAN 모드로 구성되었을 때 사용되는 포트.
      - 노드 간 Pod 트래픽을 캡슐화하여 터널링하는 데 사용됨.












 작성중 .....................................