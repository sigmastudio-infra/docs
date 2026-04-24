---
aliases:
  - Google Play 로그인 가이드
---
# Google Play 로그인 가이드

이 문서는 Unity 프로젝트에서 **Google Play 로그인을 설정하는 방법**을 정리한 문서입니다.  

-  **GCP OAuth 설정**
- **Google Play Games Services 연동**
- **사용자 인증 정보 연결**

---

## 1. GCP 에서 OAuth client 생성하기

먼저 **Google Play Console** 과 동일한 계정으로 **GCP 콘솔**에 로그인한 뒤, APIs & Services → Credentials 메뉴로 이동합니다.
![[Pasted image 20260424173409.png]]
여기서 **Create Credentials**를 눌러 **OAuth 2.0 Client ID**를 생성합니다.


생성해야 하는 항목
![[Pasted image 20260424173503.png]]
**Android**
**Web application** (게임서버)

![[Pasted image 20260424173109.png]]
OAuth 클라이언트는 2개를 생성해야 합니다.

**왜 2개를 만들어야 하나요?**

**Android OAuth Client**

모바일 기기에서 실제 앱을 인증할 때 사용합니다.
이때 패키지 이름과 **SHA-1** 인증서 지문을 기준으로 앱을 식별합니다.

**Web Application OAuth Client**

Google Play Games Services와 UGS Auth 측 인증 흐름에서 사용됩니다.

서버 인증 코드(auth code) 를 발급받아 이후 **Unity Authentication**과 연결할 때 필요합니다.


Android와 Web Application OAuth Client를 모두 생성한 뒤,  
**Google Play Console**로 이동합니다.

---
## 2. Google Play Console에서 credential

OAuth Client 생성 시 위에서 만든 Android OAuth Client를 연다.

![[Pasted image 20260424162726.png]]

입력해야 하는 값은 다음과 같습니다.

- **Package name**
- **SHA-1 certificate fingerprint**

![[Pasted image 20260424163159.png]]
![[Pasted image 20260424162550.png]]
### SHA-1 값은 어디서 가져오나요?

SHA-1 값은 위에서 만든 Android를 보고 서명 인증서 정보에서 확인한 값을 사용합니다.
![[Pasted image 20260424163159.png]]
예시:
![[Pasted image 20260423174209.png]]

> 주의: SHA-1 값이 다르면 로그인 인증이 정상적으로 동작하지 않을 수 있습니다.


같은 방식으로 OAuth Client를 하나 더 생성하되,  
이번에는 **Application type**을 `Web application`으로 선택합니다.

![[Pasted image 20260424162950.png]]

생성이 완료되면 **OAuth 2.0 Client ID**가 발급됩니다.

예시:
![[Pasted image 20260423174011.png]]

---

이 Web Client ID는 이후 **Google Play Games Services**와 연결할 때 사용됩니다.


---
## 3. 내부 테스트 진행

설정이 모두 완료되면 **내부 테스트(Internal Testing)** 를 진행해야 합니다.
내부 테스트를 진행하는 이유는 [Google Play Games 테스트 가이드](https://developer.android.com/games/pgs/test?hl=ko)를 참고하면 됩니다.

[[google-play-login-integration]] 이 문서를 참고.

---
## 4. Unity 설정

Unity에서 Google Play 로그인을 사용하려면 먼저 **Google Play Games Plugin for Unity**를 프로젝트에 적용한 뒤, 로그인 코드와 인증 코드 요청 코드를 작성해야 합니다.

### 4-1. Google Play Games Plugin 적용

[[google-play-games-plugin-for-unity]] 이 문서를 확인.
### 4-2. Google Play Games Setup  
  
Google Play Games Plugin을 import한 뒤에는 Unity에서 Google Play Games 설정을 적용해야 합니다.  
  
상단 메뉴에서 **Google Play Games → Setup → Android Setup** 메뉴로 이동합니다.  
  ![[Pasted image 20260424170112.png]]
![[Pasted image 20260424170139.png]]


이 단계에서는 Google Play Console에서 발급된 값을 Unity 프로젝트에 연결합니다.  
일반적으로 다음 정보를 입력하거나 연결하게 됩니다.  
  
![[Pasted image 20260424170256.png]]

---


### 5. 기본 로그인 흐름

[[예제 코드]]
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

    public async void SignInWithUnityAuthentication()
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

[[예제 코드]] 에서 authorization code, token 값을 출력할 수 있음