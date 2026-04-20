---
name: deploy-font-upload
description: GCS에 폰트 파일 업로드 — Cloud Run에서 시작 시 자동 다운로드됨
disable-model-invocation: true
allowed-tools: Bash(npx tsx *)
---

# /deploy:font-upload

로컬 `font/` 디렉토리의 폰트를 GCS에 업로드한다.

## 실행 단계

1. 로컬 폰트 확인: `ls font/`
2. 업로드: `npx tsx src/scripts/upload-fonts.ts`
3. GCS 확인: 6개 폰트 존재 확인

## 현재 폰트 목록
- BlackHanSans-Regular.ttf (제목, 한글+라틴)
- GmarketSansTTFBold.ttf (자막, 한글+라틴)
- GmarketSansTTFLight.ttf
- GmarketSansTTFMedium.ttf
- NotoSansCJKsc-Bold.otf (CJK: 일본어/중국어)
- NotoSansDevanagari-Bold.ttf (힌디어)

## 관련 파일
- `src/scripts/upload-fonts.ts`
- `src/storage/GoogleCloudStorageService.ts` — `FONT_FILES`, `downloadFonts()`
- `font/` — 로컬 폰트 디렉토리

## 예시
```
/deploy:font-upload
```
