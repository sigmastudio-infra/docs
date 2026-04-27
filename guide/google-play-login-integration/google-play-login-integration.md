---
aliases:
  - Google Play 로그인 가이드
---
# Google Play 로그인 가이드

이 문서는 Unity 프로젝트에서 **Google Play 로그인을 설정하는 방법**을 정리한 문서입니다.  

-  **GCP OAuth 설정**

---

## 1. GCP 에서 OAuth client 생성하기

먼저 **Google Play Console** 과 동일한 계정으로 **GCP 콘솔**에 로그인한 뒤, APIs & Services → Credentials 메뉴로 이동합니다.
![[Pasted image 20260424173409.png]]
여기서 **Create Credentials**를 눌러 **OAuth 2.0 Client ID**를 생성합니다.


생성해야 하는 항목

**Android**
**Web application** (게임 서버)

![[Pasted image 20260427142516.png|687]]
![[Pasted image 20260427142652.png]]
web application

![[Pasted image 20260427142718.png]]

android

![[Pasted image 20260427142809.png]]

SHA-1의 값은 여기서 가져옵니다.

![[Pasted image 20260427152532.png]]

![[Pasted image 20260427152555.png]]

여기에 있는 업로드 키 인증서를 SHA-1값에 삽입해줍니다.
![[Pasted image 20260427152622.png]]

![[Pasted image 20260427153306.png]]

각각 만든 모습
![[Pasted image 20260427142824.png]]

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
## 2. Play Games Services에 Android / Web OAuth Client 연결

OAuth Client 생성 시 위에서 만든 Android OAuth Client를 연다.

![[Pasted image 20260427143644.png]]
![[Pasted image 20260427143740.png]]

![[Pasted image 20260427143830.png]]

이 지문의 경우 gcp에서 만들어 둔 android OAuth에 SHA-1이 자동으로 들어갑니다.
![[Pasted image 20260424162550.png]]

![[Pasted image 20260427144047.png]]


게임서버 (web Application)의 경우 Android와 똑같고 유형만 바꿔주면 됩니다.

![[Pasted image 20260427144436.png]]

### SHA-1 값은 왜 안드로이드만 필요한가요?

SHA-1은 **Android 앱 신원 확인용**입니다.

Android는 앱을 배포할 때 **서명 인증서**로 서명합니다.  
Google은 이 앱이 진짜 등록된 앱인지 확인할 때 패키지명 + SHA-1 조합을 봅니다.

그래서 Android OAuth Client에는 필요합니다.

반대로 Web OAuth Client는 앱 서명이 없습니다. 

대신: Client ID ,Client Secret ,Redirect URI

같은 값으로 식별합니다.

---
## 3. 내부 테스트 진행

설정이 모두 완료되면 **내부 테스트(Internal Testing)** 를 진행해야 합니다.
내부 테스트를 진행하는 이유는 [Google Play Games 테스트 가이드](https://developer.android.com/games/pgs/test?hl=ko)를 참고하면 됩니다.

[[google-play-login-integration]] 이 문서를 참고.

---
## 4.  Play Console에서 Android 리소스 복사

### 4-1. Google Play Games Plugin 적용

[[google-play-games-plugin-for-unity]] 이 문서를 확인.
### 4-2. Google Play Games Setup  
  
Google Play Games Plugin을 import한 뒤에는 Unity에서 Google Play Games 설정을 적용해야 합니다.  
  
상단 메뉴에서 **Google Play Games → Setup → Android Setup** 메뉴로 이동합니다.  
  ![[Pasted image 20260427145455.png]]
현재 아무값이 없는걸 볼수있는데 이 값을 추가하기 위한 방법입니다.
  
1. **Play 게임즈 서비스 - 구성** 페이지 (**사용자 성장 > Play 게임즈 서비스 > 설정 및 관리 > 구성**에서 **리소스 가져오기**를 클릭합니다.
  
  ![[Pasted image 20260424170112.png]]
![[Pasted image 20260424170139.png]]


이 단계에서는 Google Play Console에서 발급된 값을 Unity 프로젝트에 연결합니다.  
일반적으로 다음 정보를 입력하거나 연결하게 됩니다.  


Client ID의 경우
![[Pasted image 20260427145747.png]]
![[Pasted image 20260427145816.png]]

## Unity 프로젝트에 Android 리소스 추가
![[Pasted image 20260424170256.png]]


---


### 5. 로그인 예제 코드

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

예제 코드 에서 authorization code, token 값을 출력할 수 있음