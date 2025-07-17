---
layout: single
title: "Node.js, npm, VSCode 설치 가이드 (Windows & macOS)"
categories: [Node.js, VSCode]
tags: [Install, Set-Up]
search: true
---


# Node.js, npm, VSCode 설치 가이드 (Windows & macOS)

이 강의를 따라오기 위해 필요한 개발 환경 설치 방법을 안내드립니다.  
운영체제에 따라 분리하여 설명하며, 특히 macOS는 Homebrew와 nvm을 통한 LTS 버전 설치 방법도 포함되어 있습니다.

---

## 1. Node.js 설치 방법

### ✅ Windows에서 Node.js 설치하기
1. 공식 사이트 접속: https://nodejs.org  
2. LTS 버전 다운로드 - 초록색 **"LTS"** 버튼 클릭 (예: `node-v20.x.x-lts.msi`)  
3. 설치 마법사 실행 - `Next` → `동의` → `Install` → `Finish`  
4. 명령 프롬프트에서 확인:  
   ```bash
   node -v  
   npm -v
   ```

---

### 🍎 macOS에서 Node.js 설치하기 (Homebrew)
1. `brew` 설치 확인:  
   ```bash
   brew -v
   ```  
   설치 안돼 있으면:  
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. Node.js 설치:  
   ```bash
   brew install node
   ```
3. 확인:  
   ```bash
   node -v  
   npm -v
   ```

---

### 🍎 macOS에서 LTS 버전 설치 (nvm 사용)
1. nvm 설치:  
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
   ```
2. 터미널 재시작 또는:  
   ```bash
   source ~/.nvm/nvm.sh
   ```
3. LTS 설치 및 설정:  
   ```bash
   nvm install --lts
   nvm use --lts
   nvm alias default lts/*
   ```
4. 확인:  
   ```bash
   node -v  
   npm -v
   ```

---

## 2. npm이란?

- Node.js에 포함된 **패키지 관리자**
- 외부 라이브러리 설치용 명령어 예시:  
  ```bash
  npm install react
  ```

---

## 3. VSCode 설치 방법

1. 사이트 접속: https://code.visualstudio.com/  
2. 다운로드 후 설치 및 실행

### 추천 VSCode 확장 프로그램
- ESLint: 코드 검사
- Prettier: 자동 포매팅
- Thunder Client: API 테스트
- Material Icon Theme: 아이콘 변경

---

## 🧪 설치 후 테스트

1. `index.js` 파일 생성  
2. 아래 코드 작성:
   ```javascript
   console.log("Hello, Node.js!");
   ```
3. 터미널에서 실행:
   ```bash
   node index.js
   ```

---

## ✅ 최종 체크리스트
- [ ] Node.js 설치 완료 (`node -v`)
- [ ] npm 설치 확인 (`npm -v`)
- [ ] VSCode 설치 및 실행 가능
- [ ] GitHub 계정 보유
- [ ] Postman 또는 Thunder Client 설치
- [ ] macOS는 LTS 전환 시 nvm 사용 여부 확인
