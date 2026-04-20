---
name: deploy-cloud-run
description: Cloud Run 배포 (빌드 → gcloud builds submit → 상태 확인 → 선택적 테스트)
disable-model-invocation: true
allowed-tools: Bash(*/gcloud *) Bash(gcloud *) Bash(npx tsc *) Bash(curl *)
argument-hint: [test=true] [test-lang=ja]
---

# /deploy

short-video-maker를 Cloud Run에 배포한다.

## 파라미터
- `test` — 배포 후 테스트 여부 (기본: false). true/false
- `test-lang` — 테스트 언어 (기본: en). tech 영상 테스트용

## 실행 단계

### 1. TypeScript 빌드
```bash
cd ./short-video-maker
npx tsc --project tsconfig.build.json
```
에러 시 중단.

### 2. Cloud Build 제출
```bash
gcloud \
  builds submit --config cloudbuild.yaml --project=YOUR_GCP_PROJECT
```
백그라운드로 실행. 빌드 ID 추적.

### 3. 빌드 상태 추적
```bash
gcloud builds describe {BUILD_ID} --project=YOUR_GCP_PROJECT --format="value(status)"
```
30초 간격, SUCCESS/FAILURE까지.

### 4. (선택) 배포 후 테스트
`test=true` 시:
```bash
curl -s -X POST https://YOUR_CLOUD_RUN_URL/api/tech/create \
  -H "Content-Type: application/json" \
  -d @temp/test-ja-payload.json
```
3분 대기 → 다운로드 확인.

## 관련 파일
- `cloudbuild.yaml` — Cloud Build 설정
- `gcp.Dockerfile` — Docker 이미지
- `src/scripts/upload-fonts.ts` — GCS 폰트 업로드 (배포 전 필요 시)

## 주의사항
- gcloud 경로: `gcloud` (PATH에 등록된 gcloud CLI 사용)
- 빌드 시간: ~9분
- 프로젝트: `YOUR_GCP_PROJECT`
- Docker에서 `--impersonate chrome` 사용 금지 (curl_cffi 없음)
- Docker에서 npm ffmpeg 사용 금지 (segfault) → `/usr/bin` 사용

## 예시
```
/deploy                    → 빌드 + 배포
/deploy test=true          → 빌드 + 배포 + 테스트
/deploy test-lang=ja       → 배포 후 일본어 테스트
```
