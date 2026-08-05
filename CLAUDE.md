# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Unity 게임 + 백엔드 프로젝트의 개발 문서를 모아둔 **Obsidian vault**다. 코드·빌드·테스트가 없는 순수 문서 저장소이며, 주제는 Google Play Games 로그인 연동, UGS(Unity Gaming Services) 인증, Firebase 설정, 개발 환경 구성 등이다. `.obsidian/` 설정은 gitignore되어 커밋되지 않는다.

## Directory layout

- `guide/` — 절차형 가이드 (콘솔 설정, SDK 연동 등)
- `setup/` — 개발 환경 설정 (예: asdf로 Java 설치)
- `design/` — 설계 문서
- `governance/` — GitHub 저장소·브랜치 운영 정책

## Documentation source of truth

문서 구조, 표현, 링크와 스크린샷 규칙은 [[governance/documentation-guide/documentation-guide|사내 위키 문서 작성 가이드]]를 따른다. 문서를 작성하거나 수정하기 전에 해당 가이드를 읽고, 공통 규칙은 개별 문서가 아니라 가이드에서 관리한다.

## Git workflow

- `main` 직접 push 금지. 모든 변경은 PR로만 반영하며 **squash merge만 허용**된다 (linear history 강제).
- 브랜치 이름은 `<type>/<short-description>` (kebab-case): `feature/*`, `fix/*`, `hotfix/*`.
- 상세 규칙은 `governance/branch-strategy.md`와 `governance/repository-settings.md` 참고.
