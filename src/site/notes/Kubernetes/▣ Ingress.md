---
{"dg-publish":true,"permalink":"/kubernetes/ingress/"}
---


Ingress는 쿠버네티스 클러스터 내의 서비스에 대한 외부 HTTP/HTTPS 트래픽을 관리하는 API 객체입니다. 서비스로의 URL 경로 기반 라우팅, SSL/TLS 종료, 이름 기반 가상 호스팅 등을 제공합니다.

## 특징

- HTTP/HTTPS 기반 라우팅 규칙 제공
- 하나의 IP 주소로 여러 서비스 노출 가능
- URL 경로 기반 라우팅
- 이름 기반 가상 호스팅
- SSL/TLS 종료
- 세션 어피니티, 리다이렉션, 커스텀 헤더 등 지원 (인그레스 컨트롤러에 따라 다름)

## 인그레스 컨트롤러

- Ingress 규칙을 구현하는 실제 컴포넌트
- 쿠버네티스에 기본으로 포함되지 않음
- 다양한 인그레스 컨트롤러 제공: Nginx, Traefik, HAProxy, AWS ALB, GCP GLBC 등
- 인그레스 정의에 따라 로드 밸런서, 리버스 프록시 등을 구성

## 클라우드 네트워크 개념과 비교

- **애플리케이션 로드 밸런서**: AWS ALB, Google Cloud Load Balancer, Azure Application Gateway와 유사
- **API 게이트웨이**: API 관리 및 라우팅 기능과 유사
- **CDN/웹 애플리케이션 방화벽**: 엣지 보안 및 라우팅 서비스와 유사한 용도로 확장 가능

## 기본 Ingress 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

## 호스트 기반 라우팅 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-routing
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

## 경로 기반 라우팅 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-routing
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

## TLS 구성 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
      - secure.example.com
    secretName: tls-secret  # 인증서와 키가 포함된 Secret
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: secure-service
            port:
              number: 80
```

## 인그레스 어노테이션 예시 (Nginx 인그레스 컨트롤러)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-with-annotations
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /app/(.*)
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

## Path 타입

- **Prefix**: 경로 프리픽스와 일치하는 URL 요청 처리 (예: /api는 /api, /api/, /api/v1 모두 매치)
- **Exact**: 정확히 일치하는 URL 경로만 처리 (예: /api는 정확히 /api만 매치)
- **ImplementationSpecific**: 인그레스 컨트롤러 구현에 따른 매칭 규칙 사용