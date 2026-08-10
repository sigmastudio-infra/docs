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
    -> Unity Authentication player 생성 또는 기존 Unity Authentication player 로그인
```

Google Play Games 플러그인은 Android 기기에서 Google 계정을 인증하고, Unity Authentication에 전달할 일회성 인증 코드를 반환합니다. 클라이언트는 이 값을 `SignInWithGooglePlayGamesAsync`에 그대로 전달합니다.

> [!IMPORTANT]
> 인증 코드는 일회용 보안 값입니다. 로그, 분석 이벤트, 스크린샷, 문서에 기록하거나 노출하지 않습니다.

## 용어

| 용어 | 의미 |
| --- | --- |
| **Google Play Games player** | Google Play Games Services가 플랫폼 기능에 사용하는 플레이어 정체성입니다. 인증에 성공하면 이 정체성의 Player ID를 얻을 수 있으며, 업적·리더보드 같은 Play Games Services 기능에 사용합니다. |
| **Unity Authentication player** | Unity Authentication이 프로젝트 안에서 관리하는 player입니다. 이 문서에서는 Google Play Games의 일회성 인증 코드로 로그인하거나 새로 생성합니다. |

두 player는 같은 개념이 아닙니다. Google Play Games player는 플랫폼 정체성이고, Unity Authentication player는 UGS의 player 정체성입니다. 게임 서버 계정과 세션은 이 문서의 범위에 포함하지 않습니다.

## 구현 단계

### 1. Google Play Games 플랫폼 활성화

Android 앱 초기화 과정에서 `PlayGamesPlatform.Activate()`를 한 번 호출해 Google Play Games 플랫폼을 활성화합니다. 이 호출은 `Social.Active`를 Google Play Games 플랫폼으로 설정할 뿐, 사용자 로그인을 시도하거나 Unity Authentication에 로그인하지는 않습니다.

`Social.Active`는 Unity **Social API**의 전역 활성 플랫폼입니다. `Social.*` 형태의 공통 Social API를 호출할 때 어느 플랫폼 구현으로 처리할지를 가리킵니다. `Activate()` 후에는 해당 호출이 기본 Local 플랫폼 대신 Google Play Games 구현을 사용합니다.

이 문서의 로그인 흐름은 `PlayGamesPlatform.Instance`를 직접 사용하므로, `Social.Active` 값은 로그인 성공 여부나 UGS 인증 상태를 뜻하지 않습니다. Unity는 Social API를 지원 중단 예정으로 안내하므로, 새 로그인 코드에서는 `Social.*` 대신 Google Play Games 플러그인의 명시적 API를 사용합니다. 자세한 역할은 [Unity Social API 문서](https://docs.unity3d.com/kr/current/Manual/net-SocialAPI.html)를 참고합니다.

```csharp
PlayGamesPlatform.Activate();
```

플랫폼 활성화는 로그인 흐름과 분리합니다. 즉, 앱 시작 시점의 초기화 책임으로 두고, 아래 로그인 시도는 인증 정책이 결정한 시점에만 실행합니다.

### 2. Google Play Games 인증

로그인 흐름이 Google Play Games 인증을 시작하기로 결정한 시점에 `PlayGamesPlatform.Instance.Authenticate`를 호출합니다. 성공 여부는 `SignInStatus`로 확인합니다.

이 단계는 **설치된 Android 앱과 Google Play Games player를 인증하는 과정**입니다. [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration|1번 문서에서 구성한 Android credential]]에 연결된 OAuth Android client의 package name과 서명 인증서 SHA-1이 현재 빌드와 일치하는지 Google Play Games가 확인합니다. 일치하면 해당 앱에서 Google Play Games 기능을 사용할 **Google Play Games player**가 인증됩니다.

따라서 Android credential은 이 단계와 직접 관련이 있습니다. package name 또는 SHA-1이 다르면 이 인증이 실패할 수 있습니다. 이 단계는 아직 Unity Authentication에 전달할 인증 코드를 발급하지 않습니다.

### 3. 일회성 인증 코드 요청

2단계가 성공한 뒤 `PlayGamesPlatform.Instance.RequestServerSideAccess`를 호출해 **일회성 인증 코드**를 요청합니다. 이 호출은 앱이나 Google Play Games player를 다시 인증하는 것이 아니라, 이미 인증된 Google Play Games 세션을 근거로 Unity Authentication에 전달할 코드를 받는 과정입니다.

이 코드의 대상은 Android credential이 아니라 Game server credential에 연결된 **Web application OAuth client**입니다. [[guide/google-play-games-login-integration/2-google-play-games-plugin-for-unity/2-google-play-games-plugin-for-unity|2번 문서의 Google Play Games 플러그인 설정]]에 입력한 Web App Client ID가 그 대상을 지정합니다. 반환된 코드는 저장하거나 로그에 남기지 않고, 다음 단계에서 Unity Authentication에 한 번만 전달합니다.

### 4. Unity Authentication 로그인

전달받은 인증 코드로 `AuthenticationService.Instance.SignInWithGooglePlayGamesAsync`를 호출합니다. 연결된 Unity Authentication player가 없으면 생성되고, 이미 연결된 Unity Authentication player가 있으면 해당 player로 로그인합니다.

```csharp
await AuthenticationService.Instance.SignInWithGooglePlayGamesAsync(authCode);
```

### 5. 실패 처리

실패는 발생 지점에 따라 구분합니다. 다음 단계로 진행할 수 있는 조건은 이전 단계가 성공한 경우뿐입니다.

1. **Google Play Games 인증 실패**: `Authenticate` 콜백의 `SignInStatus`가 `Success`가 아니면 Google Play Games player 인증이 완료되지 않은 것입니다. 이때는 인증 코드 요청과 Unity Authentication 로그인을 호출하지 않습니다.
2. **인증 코드 요청 실패**: `RequestServerSideAccess`가 비어 있는 코드를 반환하거나 요청에 실패하면 Unity Authentication에 전달할 값이 없습니다. 코드를 임의로 재사용하거나 대체하지 않습니다.
3. **Unity Authentication 로그인 실패**: `SignInWithGooglePlayGamesAsync`는 `AuthenticationException` 또는 `RequestFailedException`을 반환할 수 있습니다. 호출한 쪽은 오류 코드를 기준으로 실패를 분류하고, 인증 코드나 UGS 토큰 같은 보안 값은 로그에 남기지 않습니다.

## 구현 시 확인할 사항

- Android에서만 Google Play Games 로그인 버튼 또는 자동 로그인 흐름을 노출합니다.
- `RequestServerSideAccess`가 반환한 인증 코드는 즉시 Unity Authentication 호출에 사용하고 저장하지 않습니다.
- Google Play Games 인증을 시작하는 조건은 클라이언트 인증 설계에 따릅니다.
- 구현이 포함된 서명 AAB를 만든 뒤 [[guide/google-play-games-login-integration/5-google-play-games-internal-testing/5-google-play-games-internal-testing|내부 테스트 트랙과 실기기에서 로그인 결과를 검증합니다.]]

## 참고자료

- [Unity Authentication: Google Play Games](https://docs.unity.com/en-us/authentication/platform-signin/google-play-games)
