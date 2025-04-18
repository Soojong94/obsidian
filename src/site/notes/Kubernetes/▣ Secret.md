---
{"dg-publish":true,"permalink":"/kubernetes/secret/"}
---


Secret은 암호, API 키, 인증서와 같은 민감한 정보를 저장하기 위한 쿠버네티스 리소스입니다. ConfigMap과 유사하지만 보안이 필요한 데이터를 위해 설계되었습니다.

## 특징

- 민감한 데이터 저장 (Base64로 인코딩)
- 메모리에 저장되어 노드 디스크에 기록되지 않음
- 네임스페이스에 종속적
- 환경 변수나 볼륨으로 Pod에 마운트 가능
- 접근 제어를 위한 RBAC(Role-Based Access Control) 지원
- 다양한 타입 지원: Opaque(기본), kubernetes.io/tls, kubernetes.io/dockerconfigjson 등

## 주의사항

- 기본적으로 Secret은 Base64 인코딩만 적용되어 저장 (암호화 아님)
- 실제 암호화를 위해서는 etcd 암호화 설정 필요
- 클러스터 관리자만 Secret에 접근할 수 있도록 RBAC 설정 권장

## 클라우드 네트워크 개념과 비교

- **비밀 관리 서비스**: AWS Secrets Manager, Azure Key Vault, Google Secret Manager와 유사한 역할
- **자격 증명 저장소**: 클라우드 환경의 IAM 자격 증명 저장소와 비슷한 개념
- **인증서 관리**: SSL/TLS 인증서를 관리하는 서비스와 유사

## 실습 예시

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=  # admin (Base64 인코딩)
  password: UGEkJHcwcmQ=  # Pa$w0rd (Base64 인코딩)
```

또는 stringData 필드를 사용하여 인코딩하지 않고 직접 값 입력:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  username: admin
  password: Pa$w0rd
```

## 환경 변수로 Secret 사용 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-app
spec:
  containers:
  - name: db-app
    image: myapp:1.0
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

## 볼륨으로 Secret 사용 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-app-volume
spec:
  containers:
  - name: db-app
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
```

## Secret 생성 방법 (명령적 방식)

```bash
# 리터럴 값으로 생성
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=Pa$w0rd

# 파일에서 생성
kubectl create secret generic tls-certs \
  --from-file=cert.pem \
  --from-file=key.pem

# TLS Secret 생성
kubectl create secret tls tls-secret \
  --cert=cert.pem \
  --key=key.pem
```

## Secret 타입

- **Opaque (기본)**: 임의의 사용자 정의 데이터
- **kubernetes.io/tls**: TLS 인증서 및 키
- **kubernetes.io/dockerconfigjson**: Docker 레지스트리 인증 정보
- **kubernetes.io/service-account-token**: 서비스 계정 토큰
- **bootstrap.kubernetes.io/token**: 부트스트랩 토큰 데이터

## Secret 사용 모범 사례

- 최소 권한 원칙 적용 (필요한 Pod에만 Secret 마운트)
- etcd 암호화 설정
- Secret 변경 시 영향받는 Pod 재시작
- Secret 값을 로그에 출력하지 않도록 주의
- GitOps 워크플로우에서 Secret 관리를 위한 외부 도구 사용 (Sealed Secrets, Vault 등)