---
aliases:
  - Unity 클라이언트 Google Play Games 로그인 구현 가이드
---
# Unity 클라이언트 Google Play Games 로그인 구현 가이드

이 문서는 Android 클라이언트에서 Google Play Games 인증 코드를 받은 뒤 Unity Authentication에 로그인하는 구현 흐름을 설명합니다.

선행 작업으로 다음 설정을 완료합니다.

1. [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration|Play Games Services와 OAuth credential을 구성합니다.]]
2. [[guide/google-play-games-login-integration/2-google-play-games-plugin-for-unity/2-google-play-games-plugin-for-unity|Unity 프로젝트에 Google Play Games 플러그인을 설정합니다.]]
3. [[guide/google-play-games-login-integration/3-unity-authentication-google-play-games/3-unity-authentication-google-play-games|Unity Authentication에 Google Play Games 공급자를 연결합니다.]]

## 로그인 흐름

```text
Google Play Games 인증
    -> 서버 측 액세스용 일회성 인증 코드 요청
    -> Unity Authentication 로그인
    -> UGS player 생성 또는 기존 player 로그인
```

Google Play Games 플러그인은 Android 기기에서 Google 계정을 인증하고, Unity Authentication에 전달할 일회성 인증 코드를 반환합니다. 클라이언트는 이 값을 `SignInWithGooglePlayGamesAsync`에 그대로 전달합니다.

> [!IMPORTANT]
> 인증 코드는 일회용 보안 값입니다. 로그, 분석 이벤트, 스크린샷, 문서에 기록하거나 노출하지 않습니다.

## 구현 순서

### 1. Google Play Games 인증

앱 시작 정책에 따라 Google Play Games 플랫폼을 활성화한 뒤 `PlayGamesPlatform.Instance.Authenticate`를 호출합니다. 성공 여부는 `SignInStatus`로 확인합니다.

### 2. 일회성 인증 코드 요청

인증에 성공했을 때 `PlayGamesPlatform.Instance.RequestServerSideAccess`를 호출해 서버 측 액세스용 인증 코드를 요청합니다. Unity Authentication과 함께 사용할 때는 플러그인 설정에서 Web App Client ID가 설정되어 있어야 합니다.

### 3. Unity Authentication 로그인

전달받은 인증 코드로 `AuthenticationService.Instance.SignInWithGooglePlayGamesAsync`를 호출합니다. 연결된 Unity Authentication player가 없으면 생성되고, 이미 연결된 player가 있으면 해당 player로 로그인합니다.

```csharp
await AuthenticationService.Instance.SignInWithGooglePlayGamesAsync(authCode);
```

### 4. 실패 처리

Google Play Games 인증 실패, 인증 코드 요청 실패, Unity Authentication 요청 실패를 구분해 처리합니다. `AuthenticationException`과 `RequestFailedException`은 오류 코드를 기준으로 사용자에게 적절한 안내를 제공합니다.

인증이 실패했다고 자동으로 anonymous player를 새로 만들지 않습니다. anonymous player와 Google Play Games 계정을 연결하는 흐름은 사용자가 계정 업그레이드를 명시적으로 선택했을 때 `LinkWithGooglePlayGamesAsync`로 별도 처리합니다.

## 구현 시 확인할 사항

- Android에서만 Google Play Games 로그인 버튼 또는 자동 로그인 흐름을 노출합니다.
- `RequestServerSideAccess`가 반환한 인증 코드는 즉시 Unity Authentication 호출에 사용하고 저장하지 않습니다.
- 기존 UGS 인증 상태를 복원하는 정책과 Google Play Games 인증 시작 조건은 클라이언트 인증 설계에 따릅니다.
- 구현이 포함된 서명 AAB를 만든 뒤 [[guide/google-play-games-login-integration/5-google-play-games-internal-testing/5-google-play-games-internal-testing|내부 테스트 트랙과 실기기에서 로그인 결과를 검증합니다.]]

## 참고자료

- [Unity Authentication: Google Play Games](https://docs.unity.com/en-us/authentication/platform-signin/google-play-games)
