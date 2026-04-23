# Google Play 로그인 가이드

이 문서는 Unity 프로젝트에서 **Google Play 로그인을 설정하는 방법**을 정리한 문서입니다.  
주로 **GCP OAuth 설정**, **Google Play Games Services 연동**, 그리고 **사용자 인증 정보 연결** 과정을 다룹니다.

---

## 1. GCP 설정

먼저 **Google Play Console과 동일한 계정**으로 로그인한 뒤,  
**GCP → APIs & Services → Credentials** 메뉴로 이동합니다.

여기서 **Create Credentials**를 눌러 **OAuth 2.0 Client ID**를 생성합니다.

### 생성해야 하는 항목
- **Android**
- **Web application**

즉, OAuth 클라이언트는 **2개를 생성**해야 합니다.

### 왜 2개를 만들어야 하나요?

#### Android OAuth Client
모바일 기기에서 실제 앱을 인증할 때 사용합니다.  
이때 **패키지 이름**과 **SHA-1 인증서 지문**을 기준으로 앱을 식별합니다.

#### Web Application OAuth Client
Google Play Games Services와 서버 측 인증 흐름에서 사용됩니다.  
특히 **서버 인증 코드(auth code)** 를 발급받아 이후 서버 또는 Unity Authentication과 연결할 때 필요합니다.
	
---

## 2. Android OAuth Client 생성

OAuth Client 생성 시 **Application type**을 `Android`로 선택합니다.

입력해야 하는 값은 다음과 같습니다.

- **Package name**
- **SHA-1 certificate fingerprint**

### SHA-1 값은 어디서 가져오나요?
SHA-1 값은 **Google Play Console** 또는 서명 인증서 정보에서 확인한 값을 사용합니다.

예시:
![[Pasted image 20260423174209.png]]

> 주의: SHA-1 값이 다르면 로그인 인증이 정상적으로 동작하지 않을 수 있습니다.

---

## 3. Web Application OAuth Client 생성

같은 방식으로 OAuth Client를 하나 더 생성하되,  
이번에는 **Application type**을 `Web application`으로 선택합니다.

생성이 완료되면 **OAuth 2.0 Client ID**가 발급됩니다.

예시:
![[Pasted image 20260423174011.png]]

이 Web Client ID는 이후 **Google Play Games Services**와 연결할 때 사용됩니다.

---

## 4. Google Play Games Services 설정

Android와 Web Application OAuth Client를 모두 생성한 뒤,  
**Google Play Console**로 이동합니다.

경로는 다음과 같습니다.

**사용자 늘리기 → Play Games 서비스 → 설정**

이 메뉴에서 현재 프로젝트의 **사용자 인증 정보**를 확인할 수 있습니다.

여기서 **사용자 인증 정보 추가**를 눌러  
앞에서 만든 OAuth Client들을 연결합니다.

---

## 5. Unity 설정

Unity에서 Google Play 로그인을 사용하려면 먼저 **Google Play Games Plugin for Unity**를 프로젝트에 적용한 뒤, 로그인 코드와 인증 코드 요청 코드를 작성해야 합니다.

### 5-1. Google Play Games Plugin 적용

먼저 Unity 프로젝트에 **Google Play Games Plugin for Unity**를 import 합니다.  
plugin 적용이 완료되면 Unity에서 Google Play Games 관련 API를 사용할 수 있습니다.

이후 Android 빌드 설정과 Google Play Console 설정이 완료된 상태여야 정상적으로 로그인 테스트가 가능합니다.

---

### 5-2. 기본 로그인 흐름

Unity에서는 `PlayGamesPlatform.Activate()`를 먼저 호출하여 Google Play Games 플랫폼을 활성화한 뒤, `Authenticate()`를 통해 로그인합니다.

로그인에 성공하면 `RequestServerSideAccess()`를 호출하여 **authorization code**를 받아올 수 있습니다.  
이 코드는 이후 `SignInWithGooglePlayGamesAsync()`에 전달하여 Unity Authentication과 연동할 때 사용할 수 있습니다.

---
### 5-3. Google Play Games Setup  
  
Google Play Games Plugin을 import한 뒤에는 Unity에서 Google Play Games 설정을 적용해야 합니다.  
  
상단 메뉴에서 **Google Play Games → Setup → Android Setup** 메뉴로 이동합니다.  
  
이 단계에서는 Google Play Console에서 발급된 값을 Unity 프로젝트에 연결합니다.  
일반적으로 다음 정보를 입력하거나 연결하게 됩니다.  
  
- **Android 리소스 파일 또는 설정값**  
- **Google Play Games App ID**  
- **Web Client ID** (필요한 경우)  
- **Package Name 확인**  
  
설정이 완료되면 Unity 프로젝트 안에 Google Play Games 연동에 필요한 리소스 및 설정 파일이 생성됩니다.  
이 과정은 Unity 코드에서 `PlayGamesPlatform.Activate()` 및 `Authenticate()`를 정상적으로 사용하기 위한 사전 작업입니다.  
  
> 주의: 이 단계에서 사용하는 값들은 반드시 Google Play Console 및 GCP에 등록한 값과 일치해야 합니다.
> 


## 6. 내부 테스트 진행

설정이 모두 완료되면 **내부 테스트(Internal Testing)** 를 진행해야 합니다.

내부 테스트를 진행하는 이유는 아래 공식 문서를 참고하면 됩니다.

- [Google Play Games Unity 시작 가이드](https://developer.android.com/games/pgs/unity/unity-start?hl=ko)
- [Google Play Games 테스트 가이드](https://developer.android.com/games/pgs/test?hl=ko)

공식 문서에는 다음과 같은 내용이 안내되어 있습니다.

> 애플리케이션에서 Google Play 게임즈 서비스(PGS)가 올바르게 작동하는지 확인하려면 게임 변경사항을 Google Play에 게시하기 전에 Google Play 게임즈 서비스를 테스트하세요.

또한, 게임이 아직 게시되지 않은 상태에서는 다음 조건을 확인해야 합니다.

> 게임이 게시 취소된 상태인 경우 테스트 액세스 권한을 부여할 사용자 계정을 허용 목록에 추가해야 합니다. 그러지 않으면 테스터가 로그인과 같은 Google Play 게임즈 서비스 엔드포인트에 액세스하려고 할 때 OAuth 오류 및 404 오류가 발생합니다.

즉, 아직 정식 출시되지 않은 상태에서는 **내부 테스트 트랙을 생성하고**,  
해당 테스트에 사용할 **본인 이메일 계정 또는 테스트 계정**을 테스터 목록에 추가해야 합니다.

---

### 6-1. 내부 테스트 진행 방법

1. Google Play Console에서 **내부 테스트 트랙**을 생성합니다.
2. 테스트할 빌드를 업로드합니다.
3. 테스터 목록에 **본인 이메일**을 추가합니다.
4. 테스트 링크 또는 Play Store를 통해 앱을 설치합니다.
5. 앱에서 Google Play 로그인을 시도합니다.

---

### 6-2. 확인 방법

설정이 정상적으로 완료되면 로그인 이후 **authorization code** 또는 **token 값**이 출력됩니다.

즉, 아래와 같이 `RequestServerSideAccess()` 콜백에서 값이 정상적으로 내려오면 내부 테스트 설정이 올바르게 적용된 것입니다.

```csharp
PlayGamesPlatform.Instance.RequestServerSideAccess(true, code =>
{
    Debug.Log("Authorization code: " + code);
    Token = code;
});