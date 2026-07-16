---
aliases:
  - Unity Google Play Games 로그인 통합 가이드
---
# Unity Google Play Games 로그인 통합 가이드

이 문서는 Unity 프로젝트에 **Google Play Games 로그인을 통합하는 과정**을 정리한 가이드입니다.  
## 1. Play Console에서 Play Games Services 설정 시작하기

> 공식 문서: [Set up Google Play Games Services](https://developer.android.com/games/pgs/console/setup)

1장과 2장은 공식 문서의 설정 플로우를 그대로 따라갑니다.

모든 작업은 **Google Play Console**에서 시작합니다.
- Google Cloud Console 작업이 필요한 순간에는 Play Console이 열어주는 링크를 통해 Google Cloud Console로 이동합니다.

> 원문: [Add your game to the Play Console](https://developer.android.com/games/pgs/console/setup#add-game-to-play-console)

**선행 조건** — 시작하기 전에 다음이 준비되어 있어야 합니다.
- [Before you start](https://developer.android.com/games/pgs/console/setup#before-start) 참고

- **Google Play Developer 계정**: Play Console을 사용하려면 개발자 계정이 필요합니다. [Google Play Developer 계정 등록](https://support.google.com/googleplay/android-developer/answer/6112435)을 참고합니다. 
	- 팀으로 작업한다면 계정에 Play Games Services 관리 권한이 있어야 합니다([사용자 추가 및 권한 관리](https://support.google.com/googleplay/android-developer/answer/9844686) 참고).
- **Google Cloud 프로젝트 + Google Play Games Services API**: PGS 게임 프로젝트는 Google Cloud 프로젝트와 연결됩니다. Firebase 등으로 이미 쓰고 있는 프로젝트가 있다면 그 프로젝트를 그대로 사용하고, [Google Play Games Services API를 활성화](https://console.cloud.google.com/apis/api/games.googleapis.com)해 둡니다. 
	- 아직 프로젝트가 없다면 [새로 만들어](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project) 둡니다. 아래 3번 단계에서 연결할 프로젝트를 선택하게 됩니다.
- **Play Console에 게임(앱) 등록**: 앱을 만들고 게임으로 지정합니다. [Create and set up your app](https://support.google.com/googleplay/android-developer/answer/9859152)을 참고합니다.

선행 조건이 준비되었다면 다음 순서로 진행합니다.

1. Play Console에서 게임을 선택합니다.

2. **사용자 늘리기 > Play Games 서비스 > 설정 및 관리 > 구성**으로 이동합니다.

> Grow users > Play Games Services > Setup and management > Configuration

![[google-play-games-login-integration-001.png]]

3. **Which Play Games Services project do you want to use?** 화면에서 사용할 프로젝트 방식을 선택합니다.
	- **Create new Play Game Services project** — 새 PGS 프로젝트를 만들고, 연결할 Google Cloud 프로젝트를 선택합니다. 
		- 일반적인 경우 이 옵션을 선택합니다.
	- **Use an existing Play Games Services project** — 기존 PGS 프로젝트를 재사용합니다. 
		- 무료/유료 버전이나 국가별 버전처럼 여러 앱이 하나의 게임 프로젝트를 공유할 때만 사용합니다.

> [!WARNING] Cloud 프로젝트 선택이 중요합니다
> Firebase 등 게임에 이미 사용 중인 Google Cloud 프로젝트가 있다면 **반드시 그 프로젝트를 선택**해야 합니다. 다른 Google Cloud 프로젝트와 연결하면 게임이 Google API를 사용할 때 문제가 생길 수 있습니다. 화면 안내문도 같은 내용을 강조합니다.
> 
> > If you are using a Google Cloud project for your game, select that same Google Cloud project. For example, a project that you're using with Firebase services.

완료되면 Play Games Services 게임 프로젝트가 생성되고, 선택한 Google Cloud 프로젝트와 연결됩니다.

> [!QUESTION] Play Games Services와 Google Cloud 프로젝트는 어떤 관계인가요?
> 
> 1:1 관계입니다. 공식 문서는 "For setting up Play Games Services (PGS), a unique Google Cloud project is required"라고 명시합니다([Manage Play Games Services project settings in Google Cloud](https://developer.android.com/games/pgs/console/cloud-platform)). 이후 만들 OAuth client들은 모두 이 연결된 Google Cloud 프로젝트 안에 있어야 하며, 2장의 딥링크 플로우를 따르면 자동으로 그렇게 됩니다.

---
## 2. OAuth 2.0 Client ID 생성과 credential 추가

**Credential**은 OAuth 2.0 client ID와 게임을 연결(association)하는 설정입니다. 공식 문서의 정의는 다음과 같습니다.

> To set up a credential for Play Games Services, **which is the association between a client ID and your game**, use Google Cloud to create the client ID. Then, use Google Play Console to add a credential, **linking the client ID to your game**.
> 
> — [Generate an OAuth 2.0 client ID](https://developer.android.com/games/pgs/console/setup#oauth-client-id)

> [!QUESTION] 여기서 OAuth 2.0 client와 게임(your game)은 각각 무엇을 가리키나요?
> 
> **OAuth 2.0 client**는 OAuth 2.0 프로토콜의 역할 이름으로, Google 인증 서버에 사용자 인증·권한을 요청하는 **애플리케이션 자신**을 뜻합니다. "이런 애플리케이션이 Google 계정 사용자에게 로그인을 요청할 것이다"라고 Google에 등록해 둔 신원이며, 등록되는 장소가 Google Cloud 프로젝트입니다. 이 가이드에서 Android client는 Unity 게임 앱(패키지명 + SHA-1로 식별)의 신원이고, Web application client는 게임 서버(UGS Authentication / 자체 인증 서버)의 신원입니다.
> 
> **게임(your game)**은 1장에서 만든 **Play Games Services 게임 프로젝트**입니다(Play Console의 앱에 연결되어 있음).
> 
> 즉 credential은 "이 애플리케이션(OAuth client)이 이 게임(PGS 게임 프로젝트)의 정식 클라이언트다"라고 둘을 잇는 등록입니다.
> 
> 예를 들어 휴대폰에서 게임이 PGS 로그인을 시도하면, 기기에 설치된 **Google Play 서비스**(Google Play services)가 Android 시스템을 통해 요청을 보낸 앱의 패키지명과 서명 인증서(SHA-1)를 확인해 Google 서버로 전달합니다. 앱이 스스로 밝히는 값이 아니라 시스템이 확인한 값이라서 다른 앱이 사칭할 수 없습니다. Google은 이 값과 일치하는 Android OAuth client를 찾고, 그 client가 credential로 어느 PGS 게임 프로젝트에 등록되어 있는지 확인합니다. 이 대조 한 번으로 "어느 게임에 대한 로그인인지"와 "정식 배포된 앱이 보낸 요청인지"가 동시에 확인됩니다. 등록되지 않은 인증서로 서명한 빌드가 로그인에 실패하는 이유도 이 대조 과정 때문입니다.

credential은 **게임 앱용(Android)** 과 **게임 서버용(Web application)** 두 개를 만들어야 합니다. 각각이 왜 필요한지는 2-2와 2-3에서 설명합니다.

### 2-1. OAuth 동의 화면 구성

> 원문: [Configure the OAuth consent screen](https://developer.android.com/games/pgs/console/setup#config-oauth-consent)

OAuth client를 만들기 전에 **OAuth 동의 화면(consent screen)** 이 구성되어 있어야 합니다.

1. 1장을 마치고 구성 페이지(Play Games Services > Setup and management > Configuration)의 **Credentials** 섹션으로 이동하면 credential을 추가하기 전에 동의 화면을 먼저 구성하라는 안내 상자가 표시되고, 그 전까지 **Add credential** 버튼은 비활성화되어 있습니다.
	- 아래 스크린샷의 "Add credential" 버튼이 회색으로 표시
![[google-play-games-login-integration-002.png]]

2. 안내 상자의 **Configure**를 클릭하면 **Configure your OAuth consent screen** 대화상자가 열립니다.

![[google-play-games-login-integration-003.png]]

3. 대화상자에 안내된 6단계를 진행합니다. 1단계의 **Google Cloud Platform** 링크를 클릭해 이동합니다.
	- 이동하면 대화상자가 말하는 "OAuth consent screen" 페이지 대신 **Google Auth Platform > Overview** 화면이 열립니다. Google Cloud가 동의 화면 설정을 **Google Auth Platform**으로 개편했기 때문이며, 대화상자의 나머지 단계를 여기서 진행하면 됩니다.
		- 상단 프로젝트 선택기(또는 URL의 `?project=`)가 1장에서 연결한 Google Cloud 프로젝트인지 확인합니다.

![[google-play-games-login-integration-004.png]]

4. **Get started**를 클릭해 초기 구성(앱 정보, 공개 대상, 연락처)을 진행합니다. 
- 대화상자 단계들은 개편된 UI의 다음 메뉴에 대응합니다.
	- 3단계의 앱 이름 (Play Console의 게임 이름과 일치) — **Branding**
	- 2단계의 공개 범위·게시 상태 — **Audience**
	- 4단계의 scopes(`games`, `games_lite`, `drive.appdata`) 추가 — **Data Access**

![[google-play-games-login-integration-005.png]]

마법사의 **Audience** 단계에서는 **External**을 선택합니다.

> [!QUESTION] Audience의 Internal과 External은 무엇이 다른가요?
> 
> **Internal**은 같은 Google Workspace 조직에 속한 계정만 로그인할 수 있는 모드입니다. 사내 도구를 위한 것이라 게시나 테스트 사용자 등록 없이 바로 쓸 수 있지만, Workspace 조직이 없는 개인 Gmail 계정 프로젝트에서는 선택할 수 없습니다.
> 
> **External**은 모든 Google 계정 사용자가 로그인할 수 있는 모드로, Play 스토어에 배포하는 게임은 이쪽을 선택합니다. 만든 직후에는 **Testing** 상태라 등록한 테스트 사용자(최대 100명)만 로그인할 수 있고, **게시(publish)** 하면 모든 사용자에게 열립니다. PGS 대화상자의 "5. Publish your consent screen" 단계가 바로 이 게시를 말합니다.

5. 왼쪽 메뉴 **Data Access**에서 scope를 추가합니다. 
![[google-play-games-login-integration-006.png]]

Data Access 메뉴로 이동합니다.
![[google-play-games-login-integration-007.png]]

**Add or remove scopes**를 클릭하고, 목록에서 `games`, `games_lite`, `drive.appdata` 세 scope를 체크한 뒤 **Update** 합니다.
![[google-play-games-login-integration-008.png]]
**Save** 로 저장합니다.

6. Google Cloud Console에서 구성을 마치면 Play Console로 돌아와 **Confirm configuration**을 클릭합니다. 
![[google-play-games-login-integration-009.png]]

7. **Add credential** 버튼이 활성화되면 credential을 만들 준비가 된 것입니다.
	- 회색으로 보이던 "Add credential" 버튼이 파란색으로 변한 것을 확인하면 됩니다.
![[google-play-games-login-integration-010.png]]

### 2-2. Android credential 만들기

> 원문: [Create a credential – Android](https://developer.android.com/games/pgs/console/setup#android)

Android credential은 **모바일 기기에서 게임 앱 자체를 인증**하기 위해 필요합니다. Google은 패키지 이름과 **SHA-1** 인증서 지문으로 어떤 앱이 보낸 요청인지 식별합니다.

1. **Credentials** 섹션에서 **Add credential**을 클릭하고, 유형으로 **Android**를 선택합니다.
![[google-play-games-login-integration-011.png]]

2. **Name** 필드가 게임 이름과 일치하는지 확인합니다.
3. **Authorization** 단계에서 **Create OAuth client**를 클릭하면 **How to create OAuth client** 대화상자가 열립니다. Google Cloud 폼에 입력할 값(Type, Name, Fingerprint, Package name)이 여기에 미리 준비되어 표시됩니다.
![[google-play-games-login-integration-012.png]]

4. 대화상자의 **Create OAuth Client ID** 링크를 클릭해 Google Cloud의 **Create OAuth client ID** 폼으로 이동합니다. 딥링크로 열면 아래 값들이 미리 채워져 있으므로, 대화상자의 값과 일치하는지 확인합니다.
	- **Application type**: Android
	- **Name**: 대화상자의 Name
	- **Package name**: 대화상자의 Package name
	- **SHA-1 certificate fingerprint**: 대화상자의 Fingerprint
![[google-play-games-login-integration-013.png]]

> [!QUESTION] 대화상자의 Fingerprint는 어디서 오나요?
> 
> Play Console의 **App signing key certificate** 항목에 표시되는 SHA-1 값입니다. 이 값이 무엇인지는 Play 스토어의 앱 서명 체계를 알면 명확해집니다.
> 
> Android는 서명되지 않은 앱을 설치할 수 없고, 서명에 쓰인 키가 곧 앱의 신원이 됩니다. 현재 Play 스토어는 **Play App Signing** 체계를 씁니다. 개발자는 **업로드 키(upload key)** 로 서명한 파일을 Play Console에 올리고 — 이 서명은 업로드가 진짜 개발자에게서 왔는지 확인하는 용도입니다 — Google은 자신이 보관하는 **앱 서명 키(app signing key)** 로 **스토어 배포본**(사용자가 스토어에서 내려받아 기기에 설치하는 APK)을 다시 서명합니다. 개발자가 키를 직접 보관하다 분실·유출하는 사고를 막기 위해 Google이 최종 서명을 대신하는 구조입니다. 따라서 로그인 검증 때 기기에서 확인되는 서명 주체는 앱 서명 키이고, OAuth client에 등록할 지문도 이 키 기준이어야 합니다.
> 
> **키로 서명한다면서 왜 인증서가 나오나요?** 앱 서명은 공개키 암호 방식입니다. 서명은 **개인 키**로 만들고, 검증은 짝이 되는 **공개 키**로 합니다. **인증서(certificate)** 는 그 공개 키를 담아 배포본에 함께 포함되는 문서입니다. 개인 키는 Google 서버에만 있고, 콘솔에 표시되는 것은 공개되어도 되는 인증서 쪽입니다.
> 
> **SHA-1은 해시값인가요?** 네. **지문(fingerprint)** 은 인증서 전체를 SHA-1 해시로 요약한 값으로, 인증서를 통째로 비교하는 대신 짧은 값으로 같은 인증서인지 식별하는 용도입니다. App signing 페이지에 MD5·SHA-1·SHA-256 지문이 나란히 표시되는 것도 같은 인증서를 서로 다른 해시 알고리즘으로 요약한 것입니다.
> 
> 확인 위치는 Google Cloud 폼 안내문이 가리키는 **Protected with Play > Play Store protection > Manage Play app signing**(화면 제목은 **App signing**)이며, 아래 스크린샷의 SHA-1이 대화상자의 Fingerprint와 일치합니다. 
> 
> 같은 페이지의 **Upload key certificate**(업로드 키 인증서)와 혼동하지 않도록 주의합니다. 스토어를 거치지 않는 빌드(로컬 APK 등)의 지문은 `keytool -keystore <키스토어 경로> -list`로 직접 추출합니다([Google Play 앱 서명](https://support.google.com/googleplay/android-developer/answer/9842756) 참고).

![[google-play-games-login-integration-014.png]]
> [!TIP] 개발(debug) 빌드용 credential도 필요합니다
> 에디터에서 직접 빌드해 기기에 설치하는 개발 빌드는 Google의 앱 서명 키가 아니라 **개발 키스토어**(예: `~/.android/debug.keystore` 또는 Unity에 지정한 키스토어)로 서명됩니다. 이 빌드로 로그인을 테스트하려면 해당 인증서용 credential이 따로 있어야 합니다.
> 
> 방법은 위와 동일합니다. `keytool -list -keystore <키스토어 경로> -v`로 SHA-1 지문을 추출하고, **같은 패키지 이름 + 이 지문**으로 Android OAuth client를 하나 더 만든 뒤 credential로 추가합니다. 공식 문서도 release·debug 인증서로 credential을 각각 만들 것을 권장하며, 이렇게 하면 어느 인증서로 서명된 빌드든 Play Games Services가 인식합니다.
> 
> 단, Play 스토어의 내부 테스트 트랙으로 배포해 테스트하는 빌드는 앱 서명 키로 다시 서명되므로 이 credential 없이도 동작합니다.

5. Google Cloud 폼에서 **Create**를 클릭합니다. **OAuth client created** 대화상자에 Client ID가 표시되면 성공입니다. **OK**를 누릅니다.
	- 폼 하단 안내대로, 설정이 완전히 반영되기까지는 5분에서 몇 시간까지 걸릴 수 있습니다.
	
![[google-play-games-login-integration-015.png]]

6. Play Console로 돌아와 **How to create OAuth client** 대화상자의 **Done**을 클릭한 뒤, **Authorization**의 **OAuth client** 드롭다운에서 방금 만든 client를 선택합니다. 아래에 표시되는 Client ID·Fingerprint·Package name이 대화상자의 값과 같은지 확인합니다.
	- 새 client가 목록에 나타나기까지 1분쯤 걸릴 수 있습니다. 보이지 않으면 **Refresh OAuth clients**를 클릭합니다.

![[google-play-games-login-integration-016.png]]

7. **Save changes**를 클릭합니다. Configuration 페이지로 돌아가면 **Credentials > Android** 표에 credential이 **Draft** 상태로 등록된 것을 확인할 수 있습니다.
	- **Game server** 항목은 아직 비어 있습니다("You don't have any game server credentials yet"). 다음 절에서 채웁니다.

![[google-play-games-login-integration-017.png]]
### 2-3. 게임 서버 credential 만들기 (Web application)

> 원문: [Create a credential – Game server](https://developer.android.com/games/pgs/console/setup#game-server)

게임 서버 credential은 **게임 서버가 사용자를 대신해 PGS에 접근**하기 위해 필요합니다. 앱이 발급받은 서버 인증 코드(auth code)를 게임 서버 측 — 이 가이드에서는 **Unity Authentication**(UGS Auth) — 이 이 client의 Client ID/Client Secret으로 교환하면서 서버 측 인증이 이뤄집니다.

1. **Credentials** 섹션에서 **Add credential**을 다시 클릭하고, 유형으로 **Game server**를 선택합니다. **Name** 필드가 게임 이름과 일치하는지 확인합니다.

![[google-play-games-login-integration-018.png]]

2. **Authorization**의 **Create OAuth client**를 클릭하면 **How to create OAuth client** 대화상자가 열립니다. 이번에 입력할 값은 **Type: Web application**과 **Name** 두 가지뿐입니다 — 서버는 앱이 아니므로 패키지명과 지문이 없습니다. **Create OAuth Client ID** 링크를 클릭합니다.

![[google-play-games-login-integration-019.png]]

3. Google Cloud의 **Clients** 화면이 열리면 **Create client**를 클릭합니다. 목록에서 2-2에서 만든 Android client도 확인할 수 있습니다.

![[google-play-games-login-integration-020.png]]

4. **Create OAuth client ID** 폼에 **Application type**(Web application)과 **Name**이 미리 채워져 있는지 확인하고 **Create**를 클릭합니다.
	- **Authorized JavaScript origins**와 **Authorized redirect URIs**는 비워 둡니다. 우리 로그인 흐름은 브라우저 리다이렉트 없이 auth code를 교환하므로 필요하지 않습니다.

![[google-play-games-login-integration-021.png]]

5. **OAuth client created** 대화상자에 표시되는 **Client ID와 Client secret을 지금 복사해 안전한 곳에 보관합니다**. 확인 후 **OK**를 누릅니다.
	- 대화상자의 경고대로 **이 창을 닫으면 Client secret을 다시 볼 수 없습니다**. **Download JSON**으로 받아 두는 것도 방법입니다.
	- 이 Client ID와 Client secret이 아래 NOTE에서 설명하는 UGS Authentication 설정에 들어갈 값입니다.

![[google-play-games-login-integration-022.png]]

6. **Clients** 목록에 **Web application** client가 추가되어, 2-2의 Android client와 나란히 2개가 된 것을 확인합니다.

![[google-play-games-login-integration-023.png]]

7. Play Console로 돌아와 대화상자의 **Done**을 클릭하고, **OAuth client** 드롭다운에서 방금 만든 Web application client를 선택합니다. Client ID가 표시되면 **Save changes**를 클릭합니다.
	- 목록에 보이지 않으면 **Refresh OAuth clients**를 클릭합니다.

![[google-play-games-login-integration-024.png]]

8. Configuration 페이지의 **Credentials**에 **Android**와 **Game server** credential이 모두 **Draft** 상태로 등록된 것을 확인합니다. 이것으로 2장의 목표였던 credential 2개가 완성되었습니다.

![[google-play-games-login-integration-025.png]]

> [!NOTE] Web App Client ID는 뒤에서 다시 씁니다
> 여기서 만든 Web application 클라이언트의 **Client ID**는 [[#4.  Play Console에서 Android 리소스 복사해서 에디터에 넣기|4장]]에서 Unity 에디터의 **Web App Client ID** 필드에 입력하고, **Client Secret**과 함께 UGS Authentication의 ID 공급자 설정에도 사용합니다. Client Secret은 5단계의 대화상자에서만 확인할 수 있으므로, 그때 보관해 둔 값을 사용합니다.

> [!WARNING] 딥링크 대신 Google Cloud Console에서 직접 만든다면
> 반드시 **Google Play Console과 동일한 계정**으로 로그인하고, 게임의 Play Games Services에 연결된 프로젝트가 선택되어 있는지 확인하세요. 다른 프로젝트에 만든 OAuth client는 credential 추가 시 드롭다운에 나타나지 않습니다. 공식 문서도 "Google Cloud Console에서 client ID를 만들기만 하면 PGS는 게임과 client의 연결을 알지 못한다"고 설명합니다([Avoid common issues](https://developer.android.com/games/pgs/console/setup)).

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
  ![[google-play-games-login-integration-026.png]]
**Resource Definition**에 현재 아무 값이 없는 걸 볼 수 있는데 아래는 이 값을 추가하기 위한 방법입니다.

**Google Play Console >사용자 늘리기 > Play Games 서비스 >  설정 및 관리 > 설정** > **게임 서버**에서

리소스의 경우 **Google Play Console >사용자 늘리기 > Play Games 서비스 >  설정 및 관리 > 설정** > **사용자 인증 정보** > **리소스 보기**에서 찾을 수 있습니다.
    ![[google-play-games-login-integration-027.png]]

여기서 리소스를 복사해서 가져오기

![[google-play-games-login-integration-028.png]]

다음으로 **Web App Client ID**에 값을 채웁니다.

**Web App Client ID**의 경우 **Google Play Console >사용자 늘리기 > Play Games 서비스 >  설정 및 관리 > 설정** > **사용자 인증 정보** >  **게임 서버**에서 찾을 수 있습니다.

![[google-play-games-login-integration-029.png]]
![[google-play-games-login-integration-030.png]]
다음과 같이 채우게 됩니다.

![[google-play-games-login-integration-031.png]]

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

