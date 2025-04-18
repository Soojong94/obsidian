---
{"dg-publish":true,"permalink":"/Kubernetes/▣ Job과 CronJob/"}
---


Job은 완료 후 종료되는 일회성 작업을 실행하는 리소스입니다. CronJob은 스케줄에 따라 Job을 주기적으로 실행합니다.

## Job의 특징

- 특정 작업이 성공적으로 완료되도록 보장
- 장애 발생 시 재시도 가능
- 병렬 처리 지원
- 배치 작업 처리에 적합

## CronJob의 특징

- Linux crontab과 유사한 스케줄링 형식
- 백업, 리포트 생성 등 정기적인 작업에 적합
- 특정 시간에 자동으로 Job 생성

## 클라우드 네트워크 개념과 비교

- **스케줄된 작업**: AWS Lambda 스케줄링이나 Azure Functions 타이머 트리거와 유사하게 CronJob은 예약된 시간에 작업을 실행합니다.
- **배치 프로세싱**: AWS Batch나 GCP Batch와 유사하게 Job은 일회성 작업을 처리하고 완료됩니다.

## 실습 예시

### Job 정의 예시

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculator
spec:
  completions: 3      # 총 작업 완료 횟수
  parallelism: 2      # 동시에 실행할 Pod 수
  backoffLimit: 4     # 실패 시 재시도 횟수
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never  # 중요: Job에서는 OnFailure 또는 Never만 사용 가능
```

### CronJob 정의 예시

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-database
spec:
  schedule: "0 2 * * *"     # 매일 02:00에 실행
  concurrencyPolicy: Forbid # 이전 작업이 완료되지 않았다면 새 작업 건너뛰기
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: database-backup:v1
            command: ["/bin/sh", "/backup.sh"]
          restartPolicy: OnFailure
```

## Job 및 CronJob 관리

### Job 상태 확인

```bash
kubectl get jobs
kubectl describe job pi-calculator
```

### CronJob 및 생성된 Job 확인

```bash
kubectl get cronjobs
kubectl get jobs --selector=job-name=backup-database-1678532400
```

### Job 수동 종료

```bash
kubectl delete job pi-calculator
```

### CronJob 일시 중지

```bash
kubectl patch cronjobs backup-database -p '{"spec":{"suspend":true}}'
```

## 관련 리소스

- **Pod**: Job이 실제로 생성하는 실행 단위
- **ServiceAccount**: Job이 사용하는 권한 계정
- **ConfigMap/Secret**: 작업 실행에 필요한 구성 정보 저장