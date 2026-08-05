---
aliases:
  - Google Play Games Services와 OAuth credential 설정 가이드
---
# Google Play Games Services와 OAuth credential 설정 가이드

이 문서는 Play Console에서 Play Games Services 프로젝트를 설정하고, Android 앱과 게임 서버에 사용할 OAuth credential을 만드는 절차를 안내합니다.

다음 과정을 순서대로 진행합니다.

1. Play Games Services 프로젝트를 Google Cloud 프로젝트와 연결합니다.
2. OAuth 동의 화면을 구성합니다.
3. Android credential을 생성합니다.
4. 게임 서버용 Web application credential을 생성합니다.

## Play Console에서 Play Games Services 설정 시작하기

> 공식 문서: [Set up Google Play Games Services](https://developer.android.com/games/pgs/console/setup)

이 문서는 Google 공식 설정 흐름을 현재 콘솔 UI에 맞춰 설명합니다.

설정은 **Google Play Console**에서 시작합니다. Google Cloud Console에서 작업해야 할 때는 Play Console이 제공하는 링크를 사용해 이동합니다.

> 원문: [Add your game to the Play Console](https://developer.android.com/games/pgs/console/setup#add-game-to-play-console)

**선행 조건** — 시작하기 전에 다음을 준비합니다.

- [Before you start](https://developer.android.com/games/pgs/console/setup#before-start) 참고

- **Google Play Developer 계정**: Play Console을 사용하려면 개발자 계정이 필요합니다. [Google Play Developer 계정 등록](https://support.google.com/googleplay/android-developer/answer/6112435)을 참고합니다.
	- 팀으로 작업한다면 계정에 Play Games Services 관리 권한이 있어야 합니다([사용자 추가 및 권한 관리](https://support.google.com/googleplay/android-developer/answer/9844686) 참고).
- **Google Cloud 프로젝트 + Google Play Games Services API**: PGS 게임 프로젝트는 Google Cloud 프로젝트와 연결됩니다. Firebase 등으로 이미 쓰고 있는 프로젝트가 있다면 그 프로젝트를 그대로 사용하고, [Google Play Games Services API를 활성화](https://console.cloud.google.com/apis/api/games.googleapis.com)해 둡니다.
	- 아직 프로젝트가 없다면 [새로 만듭니다](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project). 아래 3번에서 연결할 프로젝트를 선택합니다.
- **Play Console에 게임(앱) 등록**: 앱을 만들고 게임으로 지정합니다. [Create and set up your app](https://support.google.com/googleplay/android-developer/answer/9859152)을 참고합니다.

준비가 끝났다면 다음 순서로 진행합니다.

1. Play Console에서 게임을 선택합니다.

2. **사용자 늘리기 > Play Games 서비스 > 설정 및 관리 > 구성**으로 이동합니다.

> Grow users > Play Games Services > Setup and management > Configuration

![[google-play-games-services-configuration-001.png]]

3. **Which Play Games Services project do you want to use?** 화면에서 사용할 프로젝트 방식을 선택합니다.
	- **Create new Play Game Services project** — 새 PGS 프로젝트를 만들고 연결할 Google Cloud 프로젝트를 선택합니다.
		- 일반적인 경우 이 옵션을 선택합니다.
	- **Use an existing Play Games Services project** — 기존 PGS 프로젝트를 재사용합니다.
		- 무료/유료 버전이나 국가별 버전처럼 여러 앱이 하나의 게임 프로젝트를 공유할 때만 사용합니다.

> [!WARNING] Cloud 프로젝트 선택이 중요합니다
> Firebase 등 게임에서 이미 사용하는 Google Cloud 프로젝트가 있다면 그 프로젝트를 선택해야 합니다. 다른 프로젝트에 연결하면 게임이 Google API를 사용할 때 문제가 생길 수 있습니다.
>
> > If you are using a Google Cloud project for your game, select that same Google Cloud project. For example, a project that you're using with Firebase services.

완료하면 Play Games Services 게임 프로젝트가 생성되고, 선택한 Google Cloud 프로젝트에 연결됩니다.

> [!QUESTION] Play Games Services와 Google Cloud 프로젝트는 어떤 관계인가요?
>
> Play Games Services 게임 프로젝트 하나에는 Google Cloud 프로젝트 하나가 연결됩니다. 공식 문서도 "For setting up Play Games Services (PGS), a unique Google Cloud project is required"라고 설명합니다([Manage Play Games Services project settings in Google Cloud](https://developer.android.com/games/pgs/console/cloud-platform)). 이후 만드는 OAuth client는 모두 이 Google Cloud 프로젝트에 만들어야 합니다. 이 문서처럼 Play Console의 링크를 통해 이동하면 연결된 프로젝트가 자동으로 선택됩니다.

## OAuth 2.0 Client ID 생성과 credential 추가

Google 공식 문서는 credential을 OAuth 2.0 client ID와 Play Games Services 게임 프로젝트를 연결하는 항목(association)으로 정의합니다. 게임이 Play Games Services와 통신하려면 승인된 OAuth client ID를 연결한 credential이 필요합니다([Generate an OAuth 2.0 client ID](https://developer.android.com/games/pgs/console/setup#oauth-client-id)).

> [!NOTE] 핵심: credential이 필요한 이유
> Google Cloud에서 OAuth client ID만 만들어서는 Play Games Services가 그 client를 어떤 게임에 사용할지 알 수 없습니다. Play Console에서 client ID를 선택해 credential을 추가해야 “이 client를 이 게임에 사용한다”는 연결이 만들어집니다. 이 연결이 없으면 게임 앱이나 게임 서버를 해당 게임 프로젝트의 client로 인증하고 권한을 부여할 수 없습니다.

### Credential 구성 관계

![[google-play-games-services-configuration-022.png]]

| Credential 종류 | OAuth client가 식별하는 대상 | 필요한 이유                                                                                                                                                                         |
| ------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Android       | Android 게임 앱          | `package name`과 SHA-1 인증서 지문이 일치하는 빌드만 Play Games Services API를 사용할 수 있습니다. 값이 다르면 인증에 실패합니다. |
| Game server   | 게임 서버 역할의 서비스         | Google Cloud에서 만든 `Web application` OAuth client를 Game server credential에 연결합니다. 앱은 이 OAuth client의 Client ID(Web App Client ID)로 일회성 server authorization code를 요청하고, Unity Authentication은 같은 Client ID와 Client Secret으로 이를 검증합니다. |

따라서 이 연동에는 Android credential과 Game server credential이 각각 필요합니다. server authorization code를 요청할 때는 Android client ID가 아니라 Game server credential에 연결한 Web App Client ID를 사용합니다([Server-side access to Google Play Games Services](https://developer.android.com/games/pgs/android/server-access), [Google Play Games sign-in with Unity Authentication](https://docs.unity.com/en-us/authentication/platform-signin/google-play-games)).

### OAuth 동의 화면 구성

> 원문: [Configure the OAuth consent screen](https://developer.android.com/games/pgs/console/setup#config-oauth-consent)

OAuth client를 만들기 전에 OAuth 동의 화면(consent screen)을 구성해야 합니다.

1. Play Games Services 게임 프로젝트를 만든 뒤 구성 페이지(**Play Games Services > Setup and management > Configuration**)의 **Credentials** 섹션으로 이동합니다. OAuth 동의 화면을 먼저 구성하라는 안내 상자가 표시되며, 구성 전에는 **Add credential** 버튼이 비활성화되어 있습니다.
	- 아래 스크린샷에서는 **Add credential** 버튼이 회색으로 표시됩니다.
![[google-play-games-services-configuration-002.png]]

2. 안내 상자의 **Configure**를 클릭하면 **Configure your OAuth consent screen** 대화상자가 열립니다.

![[google-play-games-services-configuration-003.png]]

3. 대화상자의 **Google Cloud Platform** 링크를 클릭합니다.
	- 기존 대화상자가 안내하는 **OAuth consent screen** 페이지 대신 **Google Auth Platform > Overview** 화면이 열립니다. Google Cloud가 동의 화면 설정을 Google Auth Platform으로 개편했기 때문입니다.
	- 상단 프로젝트 선택기(또는 URL의 `?project=`)가 앞에서 연결한 Google Cloud 프로젝트인지 확인합니다.
	- 기존 대화상자의 6단계는 개편된 UI에서 다음과 같이 처리합니다. 2번과 3번은 화면에 표시되는 순서만 바뀌었을 뿐, 설정 내용은 같습니다.

	| 기존 대화상자 단계 | 개편된 Google Auth Platform에서의 위치 |
	| --- | --- |
	| 1. Google Cloud Platform에서 동의 화면 설정 페이지로 이동 | 이 단계에서 연 **Google Auth Platform > Overview** |
	| 2. 공개 범위를 External 또는 Internal로 선택 | 4-2의 **Audience** |
	| 3. Play Console 게임 이름과 같은 앱 이름 입력 | 4-1의 **App Information** |
	| 4. `games`, `games_lite`, `drive.appdata` scope 추가 | 5번의 **Data Access** |
	| 5. 동의 화면 게시 | **Audience**에서 앱을 게시 |
	| 6. Play Console로 돌아가 구성 확인 | 6번의 **Confirm configuration** |

![[google-play-games-services-configuration-004.png]]

4. **Google Auth Platform > Overview** 화면에서 **Get started**를 클릭한 뒤, **Project configuration**을 다음 순서로 완료합니다.
	1. **App Information**에서 앱 이름을 Play Console의 게임 이름과 같게 입력하고 사용자 지원 이메일을 선택한 뒤 **Next**를 클릭합니다.

	![[google-play-games-services-configuration-005.png]]

	2. **Audience**에서 **External**을 선택하고 **Next**를 클릭합니다.

	![[google-play-games-services-configuration-027.png]]

	3. **Contact Information**에 연락처 이메일을 입력합니다.
	4. **Finish**에서 **Create**를 클릭해 초기 설정을 마칩니다.

> [!QUESTION] Audience의 Internal과 External은 무엇이 다른가요?
>
> **Internal**은 같은 Google Workspace 조직의 계정만 로그인할 수 있는 모드입니다. 사내 도구에는 적합하지만, Google Workspace 조직이 없는 개인 Gmail 계정 프로젝트에서는 선택할 수 없습니다.
>
> **External**은 모든 Google 계정 사용자가 로그인할 수 있는 모드입니다. Play 스토어에 배포하는 게임은 이 옵션을 선택합니다. 처음에는 **Testing** 상태이므로 등록한 테스트 사용자(최대 100명)만 로그인할 수 있습니다. 앱을 게시하면 모든 사용자에게 공개됩니다. PGS 대화상자의 **5. Publish your consent screen** 단계가 이 작업에 해당합니다.

5. 왼쪽 메뉴에서 **Data Access**를 선택해 scope를 추가합니다.
![[google-play-games-services-configuration-006.png]]

**Data Access** 메뉴가 열립니다.
![[google-play-games-services-configuration-007.png]]

**Add or remove scopes**를 클릭하고 목록에서 `games`, `games_lite`, `drive.appdata`를 선택한 뒤 **Update**를 클릭합니다.
![[google-play-games-services-configuration-008.png]]
**Save**를 클릭해 저장합니다.

6. Google Cloud Console 설정을 마치면 Play Console로 돌아와 **Confirm configuration**을 클릭합니다.
![[google-play-games-services-configuration-009.png]]

7. **Add credential** 버튼이 활성화되면 credential을 만들 수 있습니다.
	- 회색이던 **Add credential** 버튼이 파란색으로 바뀌었는지 확인합니다.
![[google-play-games-services-configuration-010.png]]

### Android credential 만들기

> 원문: [Create a credential – Android](https://developer.android.com/games/pgs/console/setup#android)

Android credential은 모바일 기기에서 실행 중인 게임 앱을 식별하는 데 사용합니다. Google은 패키지 이름과 SHA-1 인증서 지문으로 요청을 보낸 앱을 확인합니다.

1. **Credentials** 섹션에서 **Add credential**을 클릭하고, 유형으로 **Android**를 선택합니다.
![[google-play-games-services-configuration-011.png]]

2. **Name** 필드가 게임 이름과 일치하는지 확인합니다.
3. **Authorization**에서 **Create OAuth client**를 클릭합니다. **How to create OAuth client** 대화상자에 Google Cloud 폼에 입력할 값(**Type**, **Name**, **Fingerprint**, **Package name**)이 표시됩니다.
![[google-play-games-services-configuration-012.png]]

4. 대화상자의 **Create OAuth Client ID** 링크를 클릭합니다. Google Cloud의 **Create OAuth client ID** 페이지가 열리며, 아래 값이 미리 입력되어 있습니다. 대화상자에 표시된 값과 일치하는지 확인합니다.
	- **Application type**: `Android`
	- **Name**: 대화상자에 표시된 Name
	- **Package name**: 대화상자에 표시된 Package name
	- **SHA-1 certificate fingerprint**: 대화상자에 표시된 Fingerprint
![[google-play-games-services-configuration-013.png]]

> [!QUESTION] 대화상자의 Fingerprint는 어디서 오나요?
>
> Play Console의 **App signing key certificate**에 표시되는 SHA-1 값입니다. 이 값의 의미는 Play 스토어의 앱 서명 방식을 알면 이해하기 쉽습니다.
>
> Android는 서명되지 않은 앱을 설치할 수 없으므로, 서명에 사용한 키가 앱의 신원이 됩니다. Play 스토어는 **Play App Signing**을 사용합니다. 개발자는 **업로드 키(upload key)** 로 서명한 파일을 Play Console에 올리고, Google은 보관 중인 **앱 서명 키(app signing key)** 로 스토어 배포본을 다시 서명합니다. 업로드 키는 업로드한 주체를 확인하는 용도이고, 기기에 설치되는 스토어 배포본의 서명 주체는 앱 서명 키입니다. 따라서 OAuth client에는 앱 서명 키의 SHA-1 지문을 등록해야 합니다.
>
> **키로 서명하는데 왜 인증서가 필요한가요?** 앱 서명은 공개키 암호 방식을 사용합니다. 서명은 개인 키로 만들고, 검증은 짝이 되는 공개 키로 합니다. 인증서(certificate)는 이 공개 키를 담아 배포본에 포함하는 정보입니다. 개인 키는 Google 서버에 보관되고, 콘솔에는 공개해도 되는 인증서 정보가 표시됩니다.
>
> **SHA-1은 해시값인가요?** 네. 지문(fingerprint)은 인증서 전체를 SHA-1으로 요약한 해시값입니다. 인증서 전체를 비교하지 않고도 같은 인증서인지 확인할 수 있습니다. **App signing** 페이지에 MD5·SHA-1·SHA-256 지문이 함께 표시되는 이유도 같은 인증서를 서로 다른 해시 알고리즘으로 요약하기 때문입니다.
>
> 확인 위치는 Google Cloud 폼 안내문이 가리키는 **Protected with Play > Play Store protection > Manage Play app signing**입니다. 화면 제목은 **App signing**이며, 아래 스크린샷의 SHA-1이 대화상자의 **Fingerprint**와 일치합니다.
>
> 같은 페이지의 **Upload key certificate**와 혼동하지 않도록 주의합니다. 스토어를 거치지 않는 빌드(예: 로컬 APK)의 지문은 `keytool -keystore <키스토어 경로> -list`로 직접 확인합니다([Google Play 앱 서명](https://support.google.com/googleplay/android-developer/answer/9842756) 참고).

![[google-play-games-services-configuration-014.png]]
> [!TIP] 개발(debug) 빌드용 credential도 필요합니다
> 에디터에서 직접 빌드해 기기에 설치하는 개발 빌드는 Google의 앱 서명 키가 아니라 개발 키스토어(예: `~/.android/debug.keystore` 또는 Unity에 지정한 키스토어)로 서명됩니다. 이 빌드로 로그인을 테스트하려면 해당 인증서용 credential을 별도로 만들어야 합니다.
>
> 방법은 같습니다. `keytool -list -keystore <키스토어 경로> -v`로 SHA-1 지문을 확인하고, 같은 패키지 이름과 이 지문으로 Android OAuth client를 하나 더 만든 뒤 credential에 추가합니다. 공식 문서도 release·debug 인증서마다 credential을 만들 것을 권장합니다. 이렇게 하면 어느 인증서로 서명한 빌드도 Play Games Services에서 인식할 수 있습니다.
>
> 단, Play 스토어의 내부 테스트 트랙으로 배포해 테스트하는 빌드는 앱 서명 키로 다시 서명되므로 이 credential 없이도 동작합니다.

5. Google Cloud 폼에서 **Create**를 클릭합니다. **OAuth client created** 대화상자에 Client ID가 표시되면 생성이 완료된 것입니다. **OK**를 클릭합니다.
	- 폼 하단 안내처럼 설정이 완전히 반영되기까지 5분에서 몇 시간이 걸릴 수 있습니다.

![[google-play-games-services-configuration-015.png]]

6. Play Console로 돌아와 **How to create OAuth client** 대화상자에서 **Done**을 클릭합니다. **Authorization**의 **OAuth client** 목록에서 방금 만든 client를 선택하고, 표시된 Client ID·Fingerprint·Package name이 대화상자 값과 같은지 확인합니다.
	- 새 client가 목록에 나타나기까지 약 1분이 걸릴 수 있습니다. 보이지 않으면 **Refresh OAuth clients**를 클릭합니다.

![[google-play-games-services-configuration-016.png]]

7. **Save changes**를 클릭합니다. Configuration 페이지의 **Credentials > Android** 표에서 credential이 **Draft** 상태로 등록되었는지 확인합니다.
	- **Game server** 항목은 아직 비어 있습니다("You don't have any game server credentials yet"). 다음 절에서 추가합니다.

![[google-play-games-services-configuration-017.png]]
### 게임 서버 credential 만들기 (Web application)

> 원문: [Create a credential – Game server](https://developer.android.com/games/pgs/console/setup#game-server)

게임 서버 credential은 게임 서버가 사용자를 대신해 PGS에 접근할 때 필요합니다. 앱이 받은 server authorization code를 게임 서버 측에서 교환해 서버 인증을 완료합니다. 이 가이드에서는 Unity Authentication(UGS Auth)이 게임 서버 역할을 맡으며, 이 OAuth client의 Client ID와 Client Secret을 사용합니다.

1. **Credentials** 섹션에서 **Add credential**을 다시 클릭하고 **Game server**를 선택합니다. **Name**이 게임 이름과 일치하는지 확인합니다.

![[google-play-games-services-configuration-018.png]]

2. **Authorization**에서 **Create OAuth client**를 클릭합니다. **How to create OAuth client** 대화상자가 열립니다. 서버에는 패키지 이름과 인증서 지문이 없으므로 **Type**은 `Web application`으로 설정하고 **Name**만 입력하면 됩니다. **Create OAuth Client ID** 링크를 클릭합니다.

![[google-play-games-services-configuration-019.png]]

3. Google Cloud의 **Clients** 화면에서 **Create client**를 클릭합니다. 목록에서 앞에서 만든 Android client도 확인할 수 있습니다.

![[google-play-games-services-configuration-020.png]]

4. **Create OAuth client ID** 페이지에서 **Application type**이 `Web application`이고 **Name**이 입력되어 있는지 확인한 뒤 **Create**를 클릭합니다.
	- **Authorized JavaScript origins**와 **Authorized redirect URIs**는 비워 둡니다. 이 로그인 흐름은 브라우저 리다이렉트 없이 auth code를 교환하므로 필요하지 않습니다.

![[google-play-games-services-configuration-021.png]]

5. 생성이 완료되면 **OAuth client created** 대화상자에서 **Client ID**와 **Client secret**을 복사해 안전한 곳에 보관한 뒤 **OK**를 클릭합니다.
	- 대화상자를 닫으면 Client secret을 다시 볼 수 없습니다. 필요하면 **Download JSON**으로 내려받아 안전하게 보관합니다.
	- 이 Client ID와 Client secret은 아래 NOTE에서 설명하는 Unity Authentication 설정에 사용합니다.

![[google-play-games-services-configuration-026.png]]

6. **Clients** 목록에 **Web application** client가 추가되었는지 확인합니다. 앞에서 만든 Android client와 함께 두 개가 표시됩니다.

![[google-play-games-services-configuration-023.png]]

7. Play Console로 돌아와 대화상자에서 **Done**을 클릭합니다. **OAuth client** 목록에서 방금 만든 Web application client를 선택하고, Client ID가 표시되면 **Save changes**를 클릭합니다.
	- 목록에 보이지 않으면 **Refresh OAuth clients**를 클릭합니다.

![[google-play-games-services-configuration-024.png]]

8. Configuration 페이지의 **Credentials**에서 **Android**와 **Game server** credential이 모두 **Draft** 상태인지 확인합니다. 두 credential이 모두 준비되었습니다.

![[google-play-games-services-configuration-025.png]]

> [!NOTE] Web App Client ID는 다음 단계에서도 사용합니다
> 여기서 만든 Web application OAuth client의 **Client ID**는 [[guide/google-play-games-login-integration/2-google-play-games-plugin-for-unity/index|Google Play Games plugin for Unity 설정 가이드]]의 **Web App Client ID** 필드와 [[guide/google-play-games-login-integration/3-unity-authentication-google-play-games/index|Unity Authentication과 Google Play Games 연동 가이드]]의 Google Play Games 설정에 사용합니다. **Client Secret**은 Unity Authentication에만 입력합니다.

> [!WARNING] 딥링크 대신 Google Cloud Console에서 직접 만든다면
> Google Play Console과 같은 계정으로 로그인하고, 게임의 Play Games Services에 연결된 Google Cloud 프로젝트가 선택되어 있는지 확인합니다. 다른 프로젝트에 만든 OAuth client는 credential 추가 화면의 목록에 나타나지 않습니다. 공식 문서도 "Google Cloud Console에서 client ID를 만들기만 하면 PGS는 게임과 client의 연결을 알지 못한다"고 설명합니다([Avoid common issues](https://developer.android.com/games/pgs/console/setup)).


## 참고자료

- [Set up Google Play Games Services](https://developer.android.com/games/pgs/console/setup)
- [Manage Play Games Services project settings in Google Cloud](https://developer.android.com/games/pgs/console/cloud-platform)
- [Server-side access to Google Play Games Services](https://developer.android.com/games/pgs/android/server-access)
- [Google Play Games sign-in with Unity Authentication](https://docs.unity.com/en-us/authentication/platform-signin/google-play-games)
