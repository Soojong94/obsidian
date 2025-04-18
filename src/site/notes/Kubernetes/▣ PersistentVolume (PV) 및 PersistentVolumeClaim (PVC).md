---
{"dg-publish":true,"permalink":"/kubernetes/persistent-volume-pv-persistent-volume-claim-pvc/"}
---


PersistentVolume(PV)과 PersistentVolumeClaim(PVC)은 쿠버네티스에서 영구 스토리지를 관리하기 위한 리소스입니다. 이 두 리소스는 Pod의 생명주기와 독립적으로 데이터를 유지할 수 있게 해줍니다.

## 주요 개념

- **PersistentVolume (PV)**: 관리자가 프로비저닝하거나 스토리지 클래스를 통해 동적으로 프로비저닝되는 클러스터의 스토리지
- **PersistentVolumeClaim (PVC)**: 사용자가 PV에 대한 요청을 나타내는 리소스
- **StorageClass**: 관리자가 제공하는 스토리지의 "클래스"를 설명하며 동적 프로비저닝에 사용

## 특징

- Pod의 생명주기와 독립적인 데이터 저장
- 다양한 스토리지 백엔드 지원 (AWS EBS, GCE PD, Azure Disk, NFS, iSCSI 등)
- 동적 또는 정적 프로비저닝 가능
- 다양한 액세스 모드 지원:
    - ReadWriteOnce (RWO): 단일 노드에서 읽기/쓰기 가능
    - ReadOnlyMany (ROX): 여러 노드에서 읽기 전용
    - ReadWriteMany (RWX): 여러 노드에서 읽기/쓰기 가능

## 클라우드 네트워크 개념과 비교

- **블록 스토리지**: AWS EBS, Google Persistent Disk, Azure Disk와 유사
- **파일 스토리지**: AWS EFS, Google Filestore, Azure Files와 유사
- **스토리지 프로비저닝**: 클라우드에서 볼륨을 생성하고 인스턴스에 연결하는 과정과 유사
- **마운트 포인트**: VM에 스토리지를 마운트하는 것과 유사한 개념

## PersistentVolume (PV) 예시

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:  # 예시용 (프로덕션에서는 실제 스토리지 사용)
    path: /mnt/data
```

## PersistentVolumeClaim (PVC) 예시

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

## Pod에서 PVC 사용 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  containers:
  - name: web-server
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: web-data
  volumes:
  - name: web-data
    persistentVolumeClaim:
      claimName: pvc-example
```

## StorageClass 예시 (AWS EBS 사용)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
```

## PV 생명 주기

- **프로비저닝**: 정적(관리자가 직접 생성) 또는 동적(StorageClass를 통해 자동 생성)
- **바인딩**: PVC가 적절한 PV에 바인딩됨
- **사용**: Pod가 PVC를 통해 PV를 사용
- **반환**: Pod 및 PVC가 삭제된 후 처리 방식
    - Retain: PV 유지 (데이터 유지, 수동 정리 필요)
    - Delete: PV 자동 삭제 (기본 스토리지도 삭제)
    - Recycle: 기본 볼륨은 유지하고 데이터만 삭제 (deprecated)

## 스토리지 프로비저닝 방식

- **정적 프로비저닝**: 관리자가 미리 PV를 생성해 놓으면 사용자가 PVC로 요청
- **동적 프로비저닝**: StorageClass를 사용하여 PVC 요청 시 자동으로 PV 생성

## 볼륨 확장

일부 스토리지 클래스는 볼륨 확장을 지원합니다:

```yaml
# PVC의 storage 필드를 수정하여 용량 확장
spec:
  resources:
    requests:
      storage: 10Gi  # 5Gi에서 10Gi로 확장
```