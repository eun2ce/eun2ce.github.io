---
title: "[git] GitHub Actions에서 다른 Repository의 Release Asset 다운로드 방법"
date: 2025-05-12 18:47:00 +0900
categories: [ "etc", "git" ]
tags: [ "branch", "git" ]
pin: false
math: false
mermaid: false
---

## 개요
GitHub Actions 워크플로우에서 다른 repository의 release asset을 다운로드하는 방법을 설명합니다. 특히 같은 조직 내의 private repository 간 접근 방법에 대해 다룹니다.

## 방법: GitHub App 사용하기

### 1. GitHub App 생성
1. 조직 페이지 → Settings → Developer settings → GitHub Apps로 이동
2. "New GitHub App" 클릭
3. 기본 설정:
   - GitHub App name: 용도에 맞는 이름 설정 (예: "Release-Asset-Downloader")
   - Homepage URL: 조직의 GitHub URL 입력
   - Webhook: 체크 해제
4. Permissions 설정:
   - Repository permissions:
     - Contents: Read (릴리스 다운로드용)
     - Metadata: Read-only
5. Where can this GitHub App be installed?: "Only on this account" 선택
6. "Create GitHub App" 클릭

### 2. App 설치 및 인증 정보 생성
1. App 생성 후 "Install App" 클릭
2. 설치할 저장소 선택
3. App ID 확인 (General 탭에서 확인 가능)
4. Private Key 생성:
   - "Private keys" 섹션에서 "Generate a private key" 클릭
   - .pem 파일이 다운로드됨

### 3. Private Key 확인
.pem 파일 내용 확인 방법:
```bash
cat path/to/your-private-key.pem
```
또는
```bash
open -a TextEdit path/to/your-private-key.pem
```

### 4. GitHub Actions Secrets 설정
repository → Settings → Secrets and variables → Actions에서 두 개의 secret 추가:
1. `APP_ID`: GitHub App의 ID 번호
2. `APP_PRIVATE_KEY`: .pem 파일의 전체 내용 (BEGIN과 END 라인 포함)

### 5. 워크플로우 파일 작성
```yaml
name: Download Release Asset

jobs:
  download-asset:
    runs-on: ubuntu-latest
    steps:
      - name: Generate token
        id: generate_token
        uses: tibdex/github-app-token@v1
        with:
          app_id: ${{ secrets.APP_ID }}
          private_key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Download Release
        run: |
          mkdir -p temp
          gh release download v1.0.0 --repo OrgName/RepoName --pattern "*.zip" --dir temp
          unzip temp/*.zip -d temp/unzip/
          mkdir -p resources/target
          cp temp/unzip/dist/target/* resources/target/
          chmod +x resources/target/*
        env:
          GH_TOKEN: ${{ steps.generate_token.outputs.token }}
```

## 주의사항
- GitHub App의 권한은 필요한 최소한으로 설정
- Private key는 절대 공개되지 않도록 주의
- Release 버전은 실제 다운로드하고자 하는 버전으로 수정 필요
- 파일 경로와 패턴은 실제 프로젝트 구조에 맞게 수정 필요

## 장점
1. 세밀한 권한 제어 가능
2. 토큰 자동 갱신
3. 조직 레벨에서의 관리 용이
4. 보안성 향상

## 참고
- 이 방법은 2024년 3월 27일 기준으로 작성되었습니다
- GitHub의 UI나 기능은 지속적으로 업데이트될 수 있으므로, 최신 GitHub 문서를 참고하시기 바랍니다
