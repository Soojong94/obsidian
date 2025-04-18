---
{"dg-publish":true,"permalink":"/Kubernetes/▣ ConfigMap/"}
---


ConfigMap은 키-값 쌍의 형태로 구성 데이터를 저장하는 쿠버네티스 리소스입니다. 애플리케이션 코드와 구성을 분리하여 컨테이너화된 애플리케이션의 이식성을 높이는 데 사용됩니다.

## 특징

- 환경별 구성을 컨테이너 이미지와 분리
- 민감하지 않은 구성 데이터 저장
- 여러 Pod에서 재사용 가능
- 다양한 형식의 데이터 저장 가능 (단일 속성, 전체 구성 파일 등)
- 키-값 쌍, 파일, 바이너리 데이터 등을 저장

## ConfigMap 데이터 사용 방법

- **환경 변수로 사용**: Pod의 컨테이너에 환경 변수로 전달
- **명령줄 인수로 사용**: 환경 변수를 명령줄 인수에 사용
- **볼륨 마운트로 사용**: 파일 시스템에 마운트하여 파일로 접근

## 클라우드 네트워크 개념과 비교

- **매개변수 저장소**: AWS Systems Manager Parameter Store, Azure App Configuration 또는 Google Cloud Runtime Config와 유사
- **환경 구성**: 다른 환경(개발, 테스트, 운영)별로 다른 구성 파일을 사용하는 것과 유사
- **애플리케이션 설정**: 전통적인 애플리케이션 설정 파일(.properties, .yaml, .json 등)의, 클라우드 네이티브 대체제

## 실습 예시

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # 개별 속성
  database_host: "mysql.example.com"
  database_port: "3306"
  
  # 구성 파일
  app.properties: |
    color=blue
    mode=prod
    log-level=INFO
    
  # JSON 형식 설정
  app-config.json: |
    {
      "db": {
        "host": "mysql.example.com",
        "port": 3306
      },
      "cache": {
        "enabled": true,
        "ttl": 300
      }
    }
```

## 환경 변수로 ConfigMap 사용 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # 개별 환경 변수로 가져오기
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_host
    
    # ConfigMap의 모든 키를 환경 변수로 가져오기
    envFrom:
    - configMapRef:
        name: app-config
```

## 볼륨 마운트로 ConfigMap 사용 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod-volume
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

## ConfigMap 생성 방법 (명령적 방식)

```bash
# 파일에서 생성
kubectl create configmap app-config --from-file=app.properties

# 리터럴 값으로 생성
kubectl create configmap env-config --from-literal=ENV=production --from-literal=DEBUG=false

# 여러 파일이 있는 디렉토리에서 생성
kubectl create configmap config-dir --from-file=config-dir/
```

## ConfigMap 업데이트와 애플리케이션 반영

ConfigMap을 업데이트했을 때:

- 환경 변수로 사용된 경우: Pod 재시작해야 반영됨
- 볼륨으로 마운트된 경우: 자동으로 업데이트되지만, 파일 변경을 감지하도록 애플리케이션이 설계되어야 함
- 업데이트 지연: 캐시로 인해 몇 분 소요될 수 있음