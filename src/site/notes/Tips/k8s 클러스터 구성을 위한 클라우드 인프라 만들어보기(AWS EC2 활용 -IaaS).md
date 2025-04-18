---
{"dg-publish":true,"permalink":"/Tips/k8s 클러스터 구성을 위한 클라우드 인프라 만들어보기(AWS EC2 활용 -IaaS)/"}
---



Public - Bastion Host, ALB(추후 구성)  
Private - Master Node 1식 / Worker Node 2식

![](https://beta.appflowy.cloud/api/file_storage/9694f649-00d1-4d32-8f94-e01c6e535655/v1/blob/435c5064-7532-4871-a378-4fc2b899450f/tnCSN4Kix1-88ZOAZsbHH6bERohVxuDghIAjGmmiH9Q=.)
## 1. VPC 생성

### 1.1. 'VPC'만을 선택 - 최대 5개 생성 가능(24.12.02)

※ VPC 등을 선택하면 모든 인프라가 자동으로 구축됨

### 1.2. 사용할 사설 IP 대역대 선택

아래 대역대 중 택 1 하여 필요한 만큼 사용 - 최대 5개 생성 가능

- 0.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

※AWS CIDR 메뉴얼 참고

## 2. Subnet 생성

### 2.1. VPC 선택

### 2.2. 가용영역

- 고가용성을 위해 최소 2개 이상의 AZ 사용 권장
- 향후 확장성을 고려한 대역대 설정

### 2.3. 용도에 따라 Subnet 대역대 구분 생성

예시 : Public - 192.168.1.0/24 , Private - 192.168.2.0/24 , DB(mysql, redis 등) - 192.168.11.0/24

## 3. Internet G/W 및 NAT G/W 생성

### 3.1. Public Subnet 과 연결할 Internet G/W 생성

#### 3.1.1. 작업 - VPC 연결

### 3.2. Private Subnet 의 인터넷 연결을 위한 NAT G/W 생성

#### 3.2.1. NAT G/W가 위치할 Public Subnet을 선택(Private 아님!)

#### 3.2.2. 연결 유형 : 퍼블릭

#### 3.2.3. 탄력적 IP 할당

## 4. Routing Table 설정

### 4.1. Public 및 Private 라우팅을 각각 생성

#### 4.1.1. Public 용 Routing Table 설정 : 라우팅 - 라우팅 편집 - 0.0.0.0/0 대역 및 IGW 선택 - 저장

#### 4.1.2. Private 용 Routing Table 설정 : 라우팅 - 라우팅 편집 - 0.0.0.0/0 대역 및 NAT 선택 - 저장

#### 4.1.3. 필요시 로컬 라우팅 테이블도 변경(default : VPC 대역)

## 5. 네트워크 ACL 설정(NACL)

### 5.1. Public ACL

#### 5.1.1.인바운드 규칙

- SSH(22): 관리자 IP(/32)
- HTTP(80), HTTPS(443): 0.0.0.0/0
- Ephemeral(1024-65535): Private Subnet CIDR (동적 할당 포트)

#### 5.1.2. 아웃바운드 규칙

- ALL Traffic(0-65535): 0.0.0.0/0(필요시 제한하여 개방)

### 5.2. Private ACL

#### 5.2.1. 인바운드 규칙

- SSH(22): Bastion Host IP(/32)
- Kubernetes 통신:
    - kube-apiserver(6443): Master CIDR
    - etcd(2379-2380): Master CIDR
    - kubelet(10250-10252): Master/Worker CIDR
    - NodePort(30000-32767): Worker CIDR
    - Calico BGP(179): Master/Worker CIDR
    - kubelet read-only(10255): Master CIDR
- Ephemeral(1024-65535): Public Subnet CIDR (동적 할당 포트)

#### 5.2.2. 아웃바운드 규칙

- 외부 통신(0-65535): 0.0.0.0/0(필요시 제한하여 개방)
- 내부 통신: VPC CIDR

## 6. EC2 인스턴스 생성

### 6.1. Bastion Host Server

보안 접속용 서버 : 작은 크기로 만들고, 서버 관리자가 Private 서버에 접속할 경유지로 사용. 즉, Private 서버에 바로 접속할 수 없고 Bastion Host Server를 통해서 접속해야 하기 때문에 사용자에 대한 관리가 가능함.

#### 6.1.1. Public Subnet 내에 생성

#### 6.1.2. 공인IP 할당 확인

#### 6.1.3. asg(보안그룹) 설정

인바운드

- 서버 관리자 IP / SSH 접속용 22번 포트

아웃바운드

- 외부 통신(0-65535): 0.0.0.0/0(필요시 제한하여 개방)

### 6.2. Master Node / Worker Node

#### 6.2.1. Private Subnet 내에 생성

#### 6.2.2. asg(보안그룹) 설정

인바운드

- 22 (Bastion Host ip 또는 정책 선택) : SSH 접속용 22번 포트
- 6443 (k8s private asg 선택 / ex) sg-abcd1234) : kube-apiserver클러스터의 API 서버 통신용
- 2379-2380(k8s private asg 선택) : etcd server클러스터의 상태를 저장하는 etcd 통신용
- 10257 (k8s private asg 선택) : kube-controller-manager컨트롤러 매니저 통신용
- 10259 (k8s private asg 선택) : kube-scheduler스케줄러 통신용. 파드 스케줄링을 위한 내부 통신
- 10250 (k8s private asg 선택) : Kubelet API노드 관리 및 파드 운영을 위한 Kubelet 통신
- 30000-32767 (k8s private asg 선택) : NodePort 서비스 클러스터 서비스 노출용 포트 범위

아웃바운드

- 외부 통신(0-65535): 0.0.0.0/0(필요시 제한하여 개방)

## 7. Bastion Host 를 통해 접속하여 k8s 네트워크 구성 시작

[[Kubernetes/클라우드를 활용한 쿠버네티스 인프라 구축 과정\|클라우드를 활용한 쿠버네티스 인프라 구축 과정]]