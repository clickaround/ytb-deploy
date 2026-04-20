---
name: deploy-youtube-token
description: YouTube OAuth 토큰을 Secret Manager에 배포하는 워크플로우
disable-model-invocation: true
allowed-tools: Bash(gcloud secrets *) Bash(tar *) Bash(base64 *)
---

## 용도
YouTube OAuth 토큰 파일(채널별 JSON)을 GCP Secret Manager에 안전하게 저장/업데이트.

## 핵심 파일
- `src/youtube-upload/services/YouTubeSecretManager.ts` — Secret Manager 연동
- 토큰 마스터 위치: `./temp-yt/`

## 의존성
- `gcloud` CLI (인증 완료 상태)
- GCP Secret Manager API 활성화
- tar, base64 (시스템 유틸)

## 배포 단계

### 1. 토큰 파일 압축
```bash
cd ./temp-yt
tar -czvf youtube-data.tar.gz youtube-*.json
```

### 2. Base64 인코딩
```bash
base64 -w 0 youtube-data.tar.gz > youtube-data.b64.txt
```

### 3. Secret Manager에 업로드
```bash
gcloud secrets versions add YOUTUBE_DATA --data-file=youtube-data.b64.txt
```

### 4. 확인
```bash
gcloud secrets versions list YOUTUBE_DATA
```

## 토큰 파일 형식
- 파일명: `youtube-{channelName}.json` (채널별 1개)
- 내용: OAuth2 refresh_token, access_token, client_id, client_secret
- 예: `youtube-poetry.json`, `youtube-news.json`

## 다른 프로젝트에 이식하기
1. `YouTubeSecretManager.ts` 복사
2. GCP 프로젝트에 Secret Manager API 활성화
3. 서비스 계정에 `secretmanager.versions.access` 권한 부여
4. 시크릿 이름 (`YOUTUBE_DATA`)을 프로젝트에 맞게 변경

## 주의사항
- **반드시 base64 인코딩**: tar.gz는 바이너리라 직접 업로드 시 non-UTF8 에러 발생
- 이전 버전 누적 방지: 주기적으로 `gcloud secrets versions destroy` 실행
- 토큰 갱신 시 전체 tar.gz 재생성 필요 (개별 파일 업데이트 불가)
- Cloud Run에서 자동 디코딩: base64 decode → tar xzf → JSON 로드
- `.gitignore`에 `youtube-*.json`, `*.tar.gz`, `*.b64.txt` 반드시 포함
