---
aliases:
  - Google Play Games plugin for Unity 설정 가이드
---
# Google Play Games plugin for Unity 설정 가이드

이 문서는 Google Play Games plugin for Unity를 프로젝트에 적용하고 Android Setup을 완료하는 방법을 설명합니다.

1. Google Play Games plugin for Unity를 가져옵니다.
2. External Dependency Manager for Unity(EDM4U)의 중복 설치 여부를 정리합니다.
3. Android resources와 게임 서버용 Web App Client ID로 Android Setup을 완료합니다.

## 선행 조건

- [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration|Google Play Games Services와 OAuth credential 설정 가이드]]에서 Android resources와 게임 서버용 Web App Client ID를 준비합니다.

## Google Play Games Plugin 적용

이 프로젝트는 Google의 공식 v2.2.0 릴리스를 기준으로 설정합니다.

1. [공식 v2.2.0 릴리스](https://github.com/playgameservices/play-games-plugin-for-unity/releases/tag/v2.2.0)에서 소스 archive를 내려받아 압축을 풉니다.
2. 압축을 푼 폴더의 `current-build/`에서 `.unitypackage` 파일을 찾습니다.
3. Unity에서 Android를 현재 **Build Target**으로 선택합니다.
4. **Assets > Import Package > Custom Package**에서 해당 패키지를 가져옵니다.
5. import와 Android dependency resolution이 끝날 때까지 기다립니다.
6. **Window > Google Play Games** 메뉴가 생성되고 **Console**에 compile error가 없는지 확인합니다.

![[google-play-games-plugin-for-unity-007.png]]
![[google-play-games-plugin-for-unity-008.png]]

### External Dependency Manager 중복 정리

Google Play Games plugin for Unity에는 External Dependency Manager for Unity(EDM4U)가 포함되어 있습니다. 프로젝트가 이미 EDM4U를 사용한다면 import한 사본과 함께 유지하지 않습니다.

1. `Packages/manifest.json`과 `Assets/ExternalDependencyManager`를 확인해 기존 EDM4U의 설치 위치와 버전을 비교합니다.
2. 프로젝트에서 사용 중인 버전을 하나만 유지합니다. 더 새 버전을 UPM으로 관리하고 있다면 import로 추가된 `Assets/ExternalDependencyManager`를 제거합니다.
3. Unity가 다시 컴파일한 뒤 **Console**에 오류나 경고가 없는지 확인합니다.

## Google Play Games Android Setup

플러그인을 import한 뒤 **Window > Google Play Games > Setup > Android Setup**으로 이동합니다.

![[google-play-games-plugin-for-unity-001.png]]

설정 화면에는 Play Console에서 가져온 **Resource Definition**과 [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration#게임 서버 credential 만들기 (Web application)|게임 서버용 Web application credential]]의 **Client ID**를 입력합니다.

1. Play Console에서 **Grow users > Play Games Services > Setup and management > Configuration**으로 이동합니다.
2. **Credentials > View resources**를 열고 Android 리소스 정의를 복사합니다.

![[google-play-games-plugin-for-unity-003.png]]

3. 복사한 내용을 Unity 설정 창의 **Resource Definition**에 붙여 넣습니다.
4. Play Console의 **Credentials > Game server**에서 Web application credential의 Client ID를 확인합니다.

5. 해당 값을 Unity 설정 창의 **Web App Client ID**에 입력하고 **Setup**을 실행합니다. 이 필드에는 Client Secret을 입력하지 않습니다.

**Setup**을 실행하면 App ID와 Web App Client ID가 Unity 프로젝트 루트 기준 `Assets/GooglePlayGames/Resources/PlayGamesSettings.asset`에 저장됩니다.

`Assets/GPGSIds.cs`는 Resource Definition에 업적·리더보드·이벤트 등의 리소스 ID가 있을 때만 생성됩니다. 현재 프로젝트의 Resource Definition에는 `app_id`와 `package_name`만 있으므로 이 파일이 생성되지 않는 것이 정상입니다.

## 참고자료

- [Google Play Games plugin for Unity v2.2.0](https://github.com/playgameservices/play-games-plugin-for-unity/releases/tag/v2.2.0)
- [Get started with Google Play Games Services in Unity](https://developer.android.com/games/pgs/unity/unity-start)
