---
aliases:
  - Unity Authentication과 Google Play Games 연동 가이드
---
# Unity Authentication과 Google Play Games 연동 가이드

이 문서는 Google Play Games로 사용자를 인증하고 Unity Authentication에 로그인하는 방법을 설명합니다.

1. Unity Authentication에 Google Play Games ID Provider를 등록합니다.
2. Google Play Console에서 만든 Web application credential을 Unity Authentication에 연결합니다.
3. Android Setup과 Unity Authentication의 설정 값이 같은 credential을 가리키는지 확인합니다.

## 선행 조건

- [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration|게임 서버용 Web application credential 설정 가이드]]를 완료합니다.
- [[guide/google-play-games-login-integration/2-google-play-games-plugin-for-unity/2-google-play-games-plugin-for-unity|Google Play Games plugin for Unity 설정 가이드]]를 완료합니다.

## Unity Authentication에 Google Play Games ID Provider 등록

Google Play Games가 발급한 일회성 server authorization code를 Unity Authentication에 전달하려면, Google Play Console에서 만든 Web application credential을 Unity Authentication에도 등록해야 합니다.

1. Unity에서 **Edit > Project Settings > Services > Authentication**으로 이동합니다.
2. **ID Providers**에서 **Google Play Games**를 추가합니다.
3. [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration#게임 서버 credential 만들기 (Web application)|게임 서버용 Web application credential]]의 **Client ID**와 **Client Secret**을 입력합니다.
4. 저장한 뒤 Google Play Services가 활성 상태인지 확인합니다.

![[3-unity-authentication-google-play-games-001.png]]

Unity Editor에서는 Google Play Games ID Provider의 활성화 상태와 입력 필드를 확인합니다.

![[3-unity-authentication-google-play-games-002.png]]

UGS 대시보드에서는 Google Play Games의 상태가 `Enabled`인지 확인합니다.

Google Play Games Android Setup의 **Web App Client ID**와 Unity Authentication의 **Client ID**에는 같은 Web application client를 사용합니다.

## 설정 일치 확인

아래 표에서 같은 값이 관리 도구와 입력 위치 사이에 일치하는지 확인합니다.

| 확인 값 | 관리 도구와 확인 위치 | 입력 또는 연결 위치 | 확인 기준 |
| --- | --- | --- | --- |
| Android package name | Unity Editor의 Player Settings | Google Cloud Console의 Android OAuth client, Google Play Console의 앱 | 세 위치의 package name이 같습니다. |
| SHA-1 인증서 지문 | Google Play Console의 Play App Signing 또는 테스트 빌드 keystore | Google Cloud Console의 Android OAuth client | 이 문서에서는 등록 절차를 다루지 않습니다. 배포에 사용하는 인증서의 SHA-1이 Android OAuth client에 이미 등록되어 있는지만 확인합니다. |
| Play Games Services game ID와 Android resources | Google Play Console의 Play Games Services | Unity Editor의 Google Play Games Android Setup | Play Console에서 복사한 Android resources를 그대로 사용합니다. |
| Web App Client ID | Google Cloud Console의 Web application OAuth client | Unity Editor의 Google Play Games Android Setup, Unity Authentication 대시보드의 Google Play Games ID Provider | 두 입력 위치가 같은 Client ID를 사용합니다. |
| Web App Client Secret | Google Cloud Console의 Web application OAuth client | Unity Authentication 대시보드의 Google Play Games ID Provider | Unity Editor나 프로젝트 파일에는 입력하지 않습니다. |

## 참고자료

- [Google Play Games sign-in with Unity Authentication](https://docs.unity.com/en-us/authentication/platform-signin/google-play-games)
- [How Unity Authentication works](https://docs.unity.com/en-us/authentication/how-authentication-works)
