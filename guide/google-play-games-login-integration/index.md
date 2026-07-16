---
aliases:
  - Unity Google Play Games 로그인 통합 가이드
---
# Unity Google Play Games 로그인 통합 가이드

이 문서는 Unity 프로젝트에 **Google Play Games 로그인을 통합하는 과정**을 정리한 가이드입니다.  
## 1. Play Console에서 Play Games 서비스 설정 시작하기

> 공식 문서: [Set up Google Play Games Services](https://developer.android.com/games/pgs/console/setup)

1장과 2장은 공식 문서의 설정 플로우를 그대로 따라갑니다. 모든 작업은 **Google Play Console**에서 시작하며, Google Cloud Console 작업이 필요한 순간에는 Play Console이 열어주는 링크를 통해 이동합니다. 이렇게 하면 OAuth client가 항상 게임에 연결된 **Google Cloud 프로젝트** 안에 만들어집니다.

### 1-1. Play Games 서비스 구성 만들기

Play Console에 게임(앱)이 등록되어 있어야 합니다. 등록되어 있다면:

1. Play Console에서 게임을 선택합니다.
2. **사용자 늘리기 > Play Games 서비스 > 설정 및 관리 > 구성**으로 이동합니다.
3. 게임이 이미 Google API를 사용하는지 선택합니다.
	- **아니요** — 새 게임이거나 Google API를 설정한 적이 없는 경우. 게임 이름을 입력하고 만들기를 누르면 Google Cloud 프로젝트가 자동으로 생성됩니다.
	- **예** — Firebase 등 Google API를 이미 설정한 경우. Google Cloud 프로젝트 목록이 표시되며, 게임이 사용 중인 프로젝트를 선택합니다.
	- **기존 Play Games 서비스 프로젝트 사용** — 패키지명을 바꾸거나 무료/유료 버전을 나누는 특수한 경우에만 사용합니다.

> [!TODO] 📸 스크린샷
> 구성 페이지에서 Google API 사용 여부를 선택하는 화면 (3가지 옵션이 보이도록)

> [!WARNING] 이 선택은 되돌리기 어렵습니다
> 공식 문서는 잘못된 옵션을 고르면 "게임이 Google API를 사용할 때 문제를 겪을 수 있다"고 경고합니다. Firebase를 이미 쓰고 있다면 반드시 **예**를 선택하고 같은 프로젝트를 고르세요.

완료되면 Play Games 서비스 게임 프로젝트가 생성되고, 대응되는 항목이 Google Cloud Console에 만들어집니다.

> [!QUESTION] Play Games 서비스와 Google Cloud 프로젝트는 어떤 관계인가요?
> 
> 1:1 관계입니다. 공식 문서는 "For setting up Play Games Services (PGS), a unique Google Cloud project is required"라고 명시합니다([Manage Play Games Services project settings in Google Cloud](https://developer.android.com/games/pgs/console/cloud-platform)). 이후 만들 OAuth client들은 모두 이 연결된 프로젝트 안에 있어야 하며, 2장의 딥링크 플로우를 따르면 자동으로 그렇게 됩니다.

### 1-2. OAuth 동의 화면 구성

OAuth client를 만들기 전에 **OAuth 동의 화면(consent screen)** 이 구성되어 있어야 합니다.

1. **설정 및 관리 > 구성**의 **사용자 인증 정보(Credentials)** 섹션에 동의 화면을 구성하라는 메시지가 표시되면 **구성**을 클릭합니다. Google Cloud로 가는 링크가 열립니다.
2. 범위(scopes)에 `games`, `games_lite`, `drive.appdata`를 포함시킵니다. 이 세 범위는 앱 검증(verification)을 요구하지 않습니다.
3. 동의 화면은 게임이 공개되는 범위만큼 공개되어야 합니다. 공식 문서는 즉시 게시를 권장하며, 불가능하다면 테스터에게만 공개해도 로그인 테스트는 가능합니다.
4. 구성을 마치고 **완료**를 누르면 Play Console이 자동으로 새로고침되고, credential을 만들 수 있는 상태가 됩니다.

> [!TODO] 📸 스크린샷
> ① Credentials 섹션의 동의 화면 구성 안내 메시지, ② Google Cloud의 동의 화면 scopes 설정

---
## 2. Credential 추가하기 — Android / 게임 서버 OAuth Client 연결

**Credential**은 OAuth 2.0 client ID와 게임을 연결(association)하는 설정입니다. 공식 문서의 정의는 다음과 같습니다.

> To set up a credential for Play Games Services, **which is the association between a client ID and your game**, use Google Cloud to create the client ID. Then, use Google Play Console to add a credential, **linking the client ID to your game**.
> 
> — [Generate an OAuth 2.0 client ID](https://developer.android.com/games/pgs/console/setup#oauth-client-id)

credential은 2개를 만들어야 합니다.

**왜 2개를 만들어야 하나요?**

**Android OAuth Client**

모바일 기기에서 앱을 인증할 때 사용합니다. 이때 패키지 이름과 **SHA-1** 인증서 지문을 기준으로 앱을 식별합니다.

**Web Application OAuth Client**

Google Play Games Services와 UGS Auth 측 인증 흐름에서 사용됩니다.

서버 인증 코드(auth code) 를 발급받아 이후 **Unity Authentication**과 연결할 때 필요합니다.

### 2-1. Android credential 만들기

1. **사용자 인증 정보** 섹션에서 **사용자 인증 정보 추가(Add credential)** 를 클릭하고, 유형으로 **Android**를 선택합니다.
2. **이름** 필드가 게임 이름과 일치하는지 확인합니다.
3. 승인(Authorization) 단계에서 **OAuth 클라이언트 만들기(Create OAuth client)** 를 클릭합니다. 게임에 연결된 프로젝트의 Google Cloud **Create OAuth Client ID** 페이지로 가는 링크가 열립니다.
4. Google Cloud에서 다음을 입력합니다.
	- 애플리케이션 유형: **Android**
	- 이름: 게임 이름
	- 패키지 이름: Android 애플리케이션의 패키지 이름
	- **SHA-1 인증서 지문**: Play Console의 **앱 서명(App signing)** 페이지에서 복사합니다. (Play 스토어로 배포하는 경우 [Google Play 앱 서명](https://support.google.com/googleplay/android-developer/answer/9842756)의 지문을 사용)
5. **만들기**를 누른 뒤 Play Console의 대화상자에서 **완료**를 누르면 클라이언트 ID 드롭다운이 새로고침됩니다. 방금 만든 클라이언트를 선택하고 **변경사항 저장**을 누릅니다.

SHA-1 값은 Google Play Console에서 가져옵니다.

![[Pasted image 20260427152532.png]]

![[Pasted image 20260427152555.png]]

여기에 있는 업로드 키 인증서를 SHA-1값에 삽입해줍니다.
![[Pasted image 20260427152622.png]]

> [!TODO] 📸 스크린샷
> ① Add credential에서 Android 유형 선택 화면, ② Create OAuth client 대화상자(딥링크), ③ 드롭다운에서 생성한 클라이언트를 선택하고 저장하는 화면

> [!TIP] release / debug 인증서
> 공식 문서는 release 인증서와 debug 인증서 지문으로 credential을 각각 만들 것을 권장합니다. 두 credential 모두 같은 패키지 이름을 사용해야 하며, 이렇게 하면 어느 인증서로 서명된 빌드든 Play Games Services가 인식합니다.

### SHA-1 값은 왜 안드로이드만 필요한가요?

SHA-1은 **Android 앱 신원 확인용**입니다.

Android는 앱을 배포할 때 **서명 인증서**로 서명합니다.  Google은 이 앱이 진짜 등록된 앱인지 확인할 때 패키지명 + SHA-1 조합을 봅니다.

그래서 Android OAuth Client에는 필요합니다.

반대로 Web OAuth Client는 앱 서명이 없습니다. 

대신: Client ID ,Client Secret ,Redirect URI

같은 값으로 식별합니다.

### 2-2. 게임 서버 credential 만들기 (Web application)

1. **사용자 인증 정보 추가**를 다시 클릭하고, 유형으로 **게임 서버(Game server)** 를 선택합니다.
2. **이름** 필드가 게임 이름과 일치하는지 확인합니다.
3. **OAuth 클라이언트 만들기**를 클릭해 Google Cloud로 이동한 뒤 다음을 입력합니다.
	- 애플리케이션 유형: **Web application**
	- 이름: 게임 이름
4. Play Console로 돌아와 드롭다운에서 방금 만든 클라이언트를 선택하고 **변경사항 저장**을 누릅니다.

> [!TODO] 📸 스크린샷
> ① Game server 유형 선택 화면, ② Web application OAuth client 생성 화면, ③ 저장 후 credential 목록에 Android/게임 서버 2개가 나란히 보이는 화면

> [!NOTE] Web App Client ID는 뒤에서 다시 씁니다
> 여기서 만든 Web application 클라이언트의 **Client ID**는 [[#4.  Play Console에서 Android 리소스 복사해서 에디터에 넣기|4장]]에서 Unity 에디터의 **Web App Client ID** 필드에 입력하고, **Client Secret**과 함께 UGS Authentication의 ID 공급자 설정에도 사용합니다.

> [!WARNING] 딥링크 대신 Google Cloud Console에서 직접 만든다면
> 반드시 **Google Play Console과 동일한 계정**으로 로그인하고, 게임의 Play Games 서비스에 연결된 프로젝트가 선택되어 있는지 확인하세요. 다른 프로젝트에 만든 OAuth client는 credential 추가 시 드롭다운에 나타나지 않습니다. 공식 문서도 "Google Cloud Console에서 client ID를 만들기만 하면 PGS는 게임과 client의 연결을 알지 못한다"고 설명합니다([Avoid common issues](https://developer.android.com/games/pgs/console/setup)).

---
## 3. 내부 테스트 진행

설정이 모두 완료되면 **내부 테스트(Internal Testing)** 를 진행해야 합니다.

내부 테스트를 진행하는 이유는 [Google Play Games 테스트 가이드](https://developer.android.com/games/pgs/test?hl=ko)를 참고하면 됩니다.

[[guide/google-play-games-internal-testing/index|Google Play Console Internal testing 설정 가이드]] 이 문서를 참고.

---
## 4.  Play Console에서 Android 리소스 복사해서 에디터에 넣기

### 4-1. Google Play Games Plugin 적용

[[guide/google-play-games-plugin-for-unity/index|Google Play Games plugin for Unity 설정 가이드]] 이 문서를 확인.
### 4-2. Google Play Games Setup  
  
Google Play Games Plugin을 import한 뒤에는 Unity에서 Google Play Games 설정을 적용해야 합니다.  
  
상단 메뉴에서 **Google Play Games → Setup → Android Setup** 메뉴로 이동합니다.  
  ![[Pasted image 20260427145455.png]]
**Resource Definition**에 현재 아무 값이 없는 걸 볼 수 있는데 아래는 이 값을 추가하기 위한 방법입니다.

**Google Play Console >사용자 늘리기 > Play Games 서비스 >  설정 및 관리 > 설정** > **게임 서버**에서

리소스의 경우 **Google Play Console >사용자 늘리기 > Play Games 서비스 >  설정 및 관리 > 설정** > **사용자 인증 정보** > **리소스 보기**에서 찾을 수 있습니다.
    ![[Pasted image 20260424170112.png]]

여기서 리소스를 복사해서 가져오기

![[Pasted image 20260424170139.png]]

다음으로 **Web App Client ID**에 값을 채웁니다.

**Web App Client ID**의 경우 **Google Play Console >사용자 늘리기 > Play Games 서비스 >  설정 및 관리 > 설정** > **사용자 인증 정보** >  **게임 서버**에서 찾을 수 있습니다.

![[Pasted image 20260427145747.png]]
![[Pasted image 20260427145816.png]]
다음과 같이 채우게 됩니다.

![[Pasted image 20260424170256.png]]

---
## 5. 로그인 예제 코드

```csharp
using GooglePlayGames;
using GooglePlayGames.BasicApi;
using Unity.Services.Authentication;
using UnityEngine;

public class GooglePlayGamesExampleScript : MonoBehaviour
{
    public string token;
    public string error;

    private void Awake()
    {
        PlayGamesPlatform.Activate();

        PlayGamesPlatform.Instance.Authenticate(status =>
        {
            if (status == SignInStatus.Success)
            {
                Debug.Log("Login with Google Play Games successful.");

                PlayGamesPlatform.Instance.RequestServerSideAccess(true, code =>
                {
                    Debug.Log("Authorization code: " + code);
                    token = code;
                });
            }
            else
            {
                error = "Failed to retrieve Google Play Games authorization code";
                Debug.Log("Login unsuccessful");
            }
        });
    }

    public async Task SignInWithUnityAuthentication()
    {
        if (string.IsNullOrEmpty(token))
        {
            Debug.LogError("Authorization code is empty.");
            return;
        }

        await AuthenticationService.Instance.SignInWithGooglePlayGamesAsync(token);
        Debug.Log("Unity Authentication sign-in successful.");
    }
}
```
### 5-1. 확인 방법

예제 코드 에서 authorization code, token 값을 출력할 수 있음

## 참고자료

- [Set up Google Play Games Services](https://developer.android.com/games/pgs/console/setup)
- [Manage Play Games Services project settings in Google Cloud](https://developer.android.com/games/pgs/console/cloud-platform)
- [Get started with Google Play Games Services in Unity](https://developer.android.com/games/pgs/unity/unity-start)

