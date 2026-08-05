## 1. 적용 범위

 기본 적용 대상: 모든 서비스 저장소
 
 예외는 반드시 각 저장소 문서(`repositories/<repo>.md`)에 기록한다.

## 2. 브랜치 모델

기본 브랜치는 아래 5가지로 운영한다.

- `main`: 프로덕션 배포 기준 브랜치 (항상 안정 상태 유지)
- `develop`: 다음 릴리즈 통합 브랜치
- `feature/*`: 기능 개발 브랜치
- `fix/*`: 일반 버그 수정 브랜치 (긴급 아님)
- `hotfix/*`: 프로덕션 긴급 장애 수정 브랜치

## 3. 브랜치 네이밍 규칙

형식: 
- `<type>/<short-description>`
	- 소문자 + kebab-case 권장
	- 가능한 한 의미가 명확한 도메인 용어 사용

예시:
- `feature/auth-token-refresh`
- `fix/login-null-check`
- `hotfix/payment-timeout-20260415`

## 4. 작업 흐름 (기본)

1. 기능/일반 버그 작업은 `develop`에서 분기  
   - `feature/*`, `fix/*` 생성
2. 작업 완료 후 Pull Request로 `develop`에 병합
3. 릴리즈 시점에 `develop`을 `main`으로 병합
4. 태그 생성 후 배포 진행

## 5. Hotfix 흐름 (긴급 장애)

1. `main`에서 `hotfix/*` 분기
2. 수정 후 PR로 `main`에 우선 병합
3. 즉시 배포
4. **반드시 동일 변경을 `develop`에도 반영**  
   - PR 또는 merge로 역반영 누락 방지

## 6. Pull Request 및 Merge 규칙

- direct push 금지: `main`, `develop`
- 모든 변경은 PR로만 반영
- 최소 승인 리뷰 수: 1명 이상
- CI 필수 체크 통과 후 병합
- 기본 merge 방식: **Squash merge** 권장
