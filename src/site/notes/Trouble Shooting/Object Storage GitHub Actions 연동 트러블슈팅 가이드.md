---
{"dg-publish":true,"permalink":"/Trouble Shooting/Object Storage GitHub Actions 연동 트러블슈팅 가이드/"}
---


# 🔍 AWS CLI Object Storage 업로드 문제 해결 가이드

## 발생 문제

- **AWS CLI Multipart Upload 오류**
    
    - 에러 메시지: "Transfering payloads in multiple chunks using aws-chunked is not supported"
    - GitHub Actions에서 AWS CLI를 사용해 Object Storage에 파일 업로드 시 발생
- **AWS SDK 연동 시 호스트네임 문제**
    
    - 호스트 이름 앞에 *가 붙는 현상
    - SSL 인증서 검증 실패
    - 에러 메시지: "Hostname/IP does not match certificate's altnames"
- **폴더 구조 및 파일 업로드 문제**
    
    - asset 폴더 등 특정 폴더가 업로드되지 않는 문제
    - 각 파일의 경로와 권한을 개별적으로 설정해야 하는 번거로움

## 🛠️ 시도한 해결 방법

1. **AWS CLI 설정 조정**
    
    - multipart_threshold 값 조정 시도
    - 다양한 AWS CLI 설정 변경 시도
    - 결과: 동일한 오류 지속
2. **AWS SDK (Node.js) 사용**
    
    ```javascript
    const client = new S3Client({
      endpoint: 'https://storage-endpoint.example.com:443',
      credentials: {
        accessKeyId: '${{ secrets.STORAGE_ACCESS_KEY }}',
        secretAccessKey: '${{ secrets.STORAGE_SECRET_KEY }}'
      },
      region: '${{ secrets.STORAGE_REGION }}',
      forcePathStyle: true
    });
    ```
    
    - SDK 설정에서 forcePathStyle: true 추가
    - 체크섬 모드 비활성화
    - 결과: SSL 인증서 검증 문제 발생
3. **폴더 구조 관련 추가 설정**
    
    ```javascript
    async function uploadFiles(dir) {
      const files = await fs.readdir(dir, { recursive: true });
      for (const file of files) {
        if ((await fs.stat(file)).isFile()) {
          await client.send(new PutObjectCommand({
            Bucket: bucketName,
            Key: file,
            Body: await fs.readFile(file),
            ContentType: mime.getType(file) || 'application/octet-stream'
          }));
        }
      }
    }
    ```
    
    - recursive 옵션으로 하위 폴더 탐색
    - ContentType 설정 필요
    - 파일 경로 유지를 위한 추가 처리 필요

## ✨ 최종 해결 방법

**GitHub Actions Workflow 설정**

```yaml
name: Deploy to Storage

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    # AWS CLI 2.9 버전 설치
    - name: Install specific version of AWS CLI
      run: |
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64-2.9.0.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
        
    # AWS Credentials 설정
    - name: Configure AWS Credentials
      run: |
        aws configure set aws_access_key_id ${{ secrets.STORAGE_ACCESS_KEY }}
        aws configure set aws_secret_access_key ${{ secrets.STORAGE_SECRET_KEY }}
        aws configure set region ${{ secrets.STORAGE_REGION }}
        aws configure set output json
        
    # Object Storage에 파일 업로드
    - name: Upload to Storage
      run: |
        aws --endpoint-url=${{ secrets.STORAGE_URL }} s3 sync ./dist s3://${{ secrets.STORAGE_BUCKET }} --delete
```

## 🔐 환경 변수 설정 필요사항

```
STORAGE_ACCESS_KEY: 스토리지 액세스 키
STORAGE_SECRET_KEY: 스토리지 시크릿 키
STORAGE_REGION: 리전 정보
STORAGE_URL: 스토리지 엔드포인트 URL (예: https://storage-endpoint.example.com:443)
STORAGE_BUCKET: 버킷 이름
```

## 💡 주요 해결 포인트

- **AWS CLI 버전을 2.9로 다운그레이드**
- Object Storage와 호환성이 검증된 버전 사용
- 폴더 구조 유지를 위한 추가 설정 구현
- 결과: 정상적으로 파일 업로드 가능

## 🎓 체크 사항

- **S3 호환 스토리지 사용 시 호환성 확인 중요**
- 최신 버전이 항상 최선은 아님
- 검증된 버전 사용이 안정적인 운영에 도움
- **파일 업로드 시 세부 설정 확인 필요**
    - 폴더 구조
    - 파일 권한
    - Content-Type
    - 경로 유지

## 📝 참고 사항

- Object Storage는 AWS S3 호환 API 제공
- 일부 기능(예: multipart upload)에서 완벽한 호환성을 보장하지 않을 수 있음
- 연동 시 공식 문서와 호환성 매트릭스 확인 필요
- 폴더 구조와 파일 속성을 고려한 세부 설정 필요

## ⚠️ 한계점

- AWS CLI 버전을 수동으로 관리해야 함
- 향후 버전 업그레이드 시 호환성 재검증 필요
- 특정 폴더나 파일 업로드 시 추가 설정 필요할 수 있음

## 🌟 해결방안의 장점

- 검증된 AWS CLI 버전 사용으로 안정성 확보
- 폴더 구조 유지 가능
- 자동 동기화 및 정리 기능
- 설정이 간단하고 명확함