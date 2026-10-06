# Node 시작하기

> 대분류: `11-Node` / 자바스크립트 실행 환경과 버전 관리

## 설치 기록 (2026-10-07)

- Homebrew 없음 → 공식 스크립트로 nvm만 설치 (`~/.nvm`, sudo 불필요)
- `~/.zshrc`에 nvm 로드 2줄 추가 → 새 터미널에서 자동 로드
- Node LTS 설치: `nvm install --lts`

## 현재 버전

- node `v24.21.0`
- npm `11.19.0`
- 기본 별칭: `default -> lts/*`

## 앞으로 정리할 소분류

- 설치-버전관리.md — Homebrew, nvm, Node 버전 갈아끼우기
- 실행-패키지.md — node 실행, npm 기초
- 프로젝트.md — 첫 프로젝트 만들기 메모

## 버전 확인

```bash
node --version
npm --version
nvm current
```
