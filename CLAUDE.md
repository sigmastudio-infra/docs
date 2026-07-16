# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Unity 게임 + 백엔드 프로젝트의 개발 문서를 모아둔 **Obsidian vault**다. 코드·빌드·테스트가 없는 순수 문서 저장소이며, 주제는 Google Play Games 로그인 연동, UGS(Unity Gaming Services) 인증, Firebase 설정, 개발 환경 구성 등이다. `.obsidian/` 설정은 gitignore되어 커밋되지 않는다.

## Directory layout

- `guide/` — 절차형 가이드 (콘솔 설정, SDK 연동 등)
- `setup/` — 개발 환경 설정 (예: asdf로 Java 설치)
- `design/` — 설계 문서
- `governance/` — GitHub 저장소·브랜치 운영 정책

## Document conventions

- 본문은 한국어로 쓰되 기술 용어는 영문 그대로 둔다 (예: OAuth Client, credential, SHA-1).
- 카테고리에 관계없이 **문서 하나당 폴더 하나**를 쓴다. 폴더명은 kebab-case 영문이며 문서의 슬러그(사실상 제목) 역할을 한다.
  - 문서 파일명은 `index.md`다.
  - 다른 문서로의 위키링크는 전체 경로 + 표시명 형식을 쓴다: `[[guide/<슬러그>/index|한국어 제목]]`.
  - 이미지는 해당 폴더의 `images/`에 두고 Obsidian 임베드(`![[파일명.png]]`)로 삽입한다.
  - 이미지 파일명은 `<폴더명>-<번호>.png` 형식이다 (예: `firebase-project-setup-001.png`). `Pasted image <타임스탬프>.png` 형태는 붙여넣기로 생긴 임시 이름이며 이 형식으로 바꿔 나가는 중이다.
- Frontmatter: `aliases`에 한국어 제목을 넣고 H1 제목과 일치시킨다. 출처 URL은 `references`에 넣는다.
- Obsidian 문법을 적극 사용한다: 위키링크 `[[문서-슬러그]]`, 같은 문서 내 헤딩 링크 `[[#헤딩|표시명]]`, 콜아웃(`> [!QUESTION]`, `> [!WARNING]`, `> [!TIP]`, `> [!NOTE]`). `> [!TODO] 📸 스크린샷` 콜아웃은 저자가 직접 찍을 스크린샷 자리 표시다.
- 아직 내용이 없는 스텁 문서는 본문에 `TODO` 한 줄만 둔다.
- 외부 링크를 붙여넣을 때 추적 파라미터(`?_gl=...` 등)는 제거한다.
- 공식 문서(developer.android.com 등)를 근거로 쓸 때는 원문을 영어 그대로 인용하고 출처 링크를 단다.

## Git workflow

- `main` 직접 push 금지. 모든 변경은 PR로만 반영하며 **squash merge만 허용**된다 (linear history 강제).
- 브랜치 이름은 `<type>/<short-description>` (kebab-case): `feature/*`, `fix/*`, `hotfix/*`.
- 상세 규칙은 `governance/branch-strategy.md`와 `governance/repository-settings.md` 참고.
