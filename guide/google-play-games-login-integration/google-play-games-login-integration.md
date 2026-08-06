---
aliases:
  - Unity Google Play Games 로그인 연동 흐름
---
# Unity Google Play Games 로그인 연동 흐름

이 문서는 Google Play Games 로그인 결과를 Unity Authentication에 연결하기 위한 전체 흐름과 상세 가이드의 실행 순서를 요약합니다.

```text
Play Console과 Google Cloud 설정
→ Google Play Games plugin for Unity 설정
→ Unity Authentication 공급자와 인증 흐름 연결
→ Play Console 내부 테스트 및 Android 실기기 검증
```

## 진행 순서

1. [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration|Google Play Games Services와 OAuth credential을 구성합니다.]]
2. [[guide/google-play-games-login-integration/2-google-play-games-plugin-for-unity/2-google-play-games-plugin-for-unity|Unity 프로젝트에 Google Play Games 플러그인을 적용하고 Android Setup을 완료합니다.]]
3. [[guide/google-play-games-login-integration/3-unity-authentication-google-play-games/3-unity-authentication-google-play-games|Unity Authentication 공급자를 설정하고 인증 흐름을 연결합니다.]]
4. [[guide/google-play-games-login-integration/4-unity-client-google-play-games-login/4-unity-client-google-play-games-login|Unity 클라이언트에서 Google Play Games와 Unity Authentication 로그인을 구현합니다.]]
5. [[guide/google-play-games-login-integration/5-google-play-games-internal-testing/5-google-play-games-internal-testing|내부 테스트 트랙과 Android 실기기에서 연동 결과를 검증합니다.]]

## 주요 연결 값

| 값 | 생성 위치 | 사용 위치 |
| --- | --- | --- |
| Android package name | Unity Player Settings | Android OAuth client와 Play Console 앱 |
| SHA-1 인증서 지문 | Play App Signing 또는 테스트 keystore | Android OAuth client |
| Play Games Services game ID와 Android resources | Play Console | Google Play Games Android Setup |
| Web App Client ID | 게임 서버용 Web application credential | Google Play Games Android Setup과 Unity Authentication 공급자 |
| Web App Client Secret | 게임 서버용 Web application credential | Unity Authentication 공급자만 |

## 완료 상태

- Android 실기기에서 Google Play Games 인증 후 Unity Authentication 로그인이 성공합니다.
- 재실행 시 cached UGS player가 복원되어 플랫폼 로그인을 생략합니다.
- 실패하거나 사용자가 취소하면 UGS Identity를 반환하지 않습니다.
- 문서, 스크린샷, Unity Console과 Logcat에 credential 또는 token이 노출되지 않습니다.
