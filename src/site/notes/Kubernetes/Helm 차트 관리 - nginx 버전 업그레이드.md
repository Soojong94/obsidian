---
{"dg-publish":true,"permalink":"/kubernetes/helm-nginx/"}
---


Helm은 Kubernetes 애플리케이션의 패키징, 배포, 관리를 위한 패키지 관리자입니다. 복잡한 애플리케이션을 쉽게 정의하고 설치하며 업그레이드할 수 있게 해줍니다. Helm은 "Kubernetes의 apt/yum/homebrew"라고 볼 수 있습니다.

## Helm의 주요 이점

- **단순화된 배포**: 복잡한 애플리케이션을 하나의 명령으로 설치
- **일관성**: 환경 간 동일한 구성 유지
- **버전 관리**: 애플리케이션 버전 관리 및 롤백 기능
- **재사용성**: 미리 패키징된 차트 활용 가능
- **공유**: 커뮤니티와 조직 내에서 구성 공유

## 핵심 개념

- **Chart: Kubernetes 리소스를 정의하는 파일의 집합 (예: nginx 차트)**
    - 템플릿 파일, 기본값, 메타데이터 등을 포함
    - Kubernetes YAML 파일의 패키지화된 버전
- **Repository: 차트를 저장하고 공유하는 저장소 (예: bitnami 레포지토리)**
    - 공개 또는 비공개로 운영 가능
    - 여러 버전의 차트를 저장
- **Release: 클러스터에 배포된 차트의 인스턴스 (예: my-nginx)**
    - 하나의 차트로 여러 개의 릴리스 생성 가능
    - 각 릴리스는 독립적으로 관리됨

## 자주 사용하는 명령어

```bash
# 저장소 추가
helm repo add bitnami https://charts.bitnami.com/bitnami

# 저장소 정보 업데이트
helm repo update

# 릴리스 목록 확인
helm list -n <namespace>

# 차트 설치
helm install <release-name> <chart-name> -n <namespace>

# 릴리스 업그레이드
helm upgrade <release-name> <chart-name> --version <version> -n <namespace>
```

## nginx 버전 업그레이드 실습

1. 현재 설치된 nginx 확인

```bash
$ helm list -n kk-ns
NAME       NAMESPACE  REVISION  UPDATED                                STATUS    CHART         APP VERSION
kk-mock1   kk-ns      1         2025-03-10 15:20:33.472829 +0900 KST  deployed  nginx-18.1.14 1.25.3
```

2. 사용 가능한 nginx 차트 버전 확인

```bash
$ helm search repo nginx --versions
NAME            VERSION   APP VERSION  DESCRIPTION
bitnami/nginx   18.1.15   1.25.4       NGINX Open Source is a web server...
bitnami/nginx   18.1.14   1.25.3       NGINX Open Source is a web server...
...
```

3. nginx 차트 업그레이드

```bash
$ helm upgrade kk-mock1 nginx --version 18.1.15 -n kk-ns
Release "kk-mock1" has been upgraded. Happy Helming!
NAME: kk-mock1
LAST DEPLOYED: Tue Mar 11 10:25:42 2025
NAMESPACE: kk-ns
STATUS: deployed
REVISION: 2
...
```

4. 업그레이드 확인

```bash
$ helm ls -n kk-ns
NAME       NAMESPACE  REVISION  UPDATED                                STATUS    CHART         APP VERSION
kk-mock1   kk-ns      2         2025-03-11 10:25:42.683921 +0900 KST  deployed  nginx-18.1.15 1.25.4
```

## 자주 사용하는 파라미터

- **--version**: 설치/업그레이드할 차트의 버전 (예: --version 18.1.15)
- **-n/--namespace**: 리소스가 위치한 네임스페이스 (예: -n kk-ns)
- **--set**: 설치/업그레이드 시 값 설정 (예: --set replicaCount=3)
- **-f/--values**: 값 파일 지정 (예: -f custom-values.yaml)
- **--wait**: 모든 리소스가 준비될 때까지 대기
- **--atomic**: 실패 시 자동 롤백 활성화