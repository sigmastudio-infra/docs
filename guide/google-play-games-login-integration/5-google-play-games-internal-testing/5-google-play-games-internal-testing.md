---
aliases:
  - Google Play Console Internal testing 설정 가이드
---
# Google Play Console Internal testing 설정 가이드

이 문서는 Google Play Games Services 로그인 기능을 **Google Play Console 내부 테스트 트랙**으로 배포하고 Android 실기기에서 검증하는 방법을 설명합니다.

선행 작업으로 다음 설정을 완료합니다.

1. [[guide/google-play-games-login-integration/1-google-play-games-services-configuration/1-google-play-games-services-configuration|Play Games Services와 OAuth credential을 구성합니다.]]
2. [[guide/google-play-games-login-integration/2-google-play-games-plugin-for-unity/2-google-play-games-plugin-for-unity|Unity 프로젝트에 Google Play Games 플러그인을 설정합니다.]]
3. [[guide/google-play-games-login-integration/3-unity-authentication-google-play-games/3-unity-authentication-google-play-games|Unity Authentication에 Google Play Games 공급자를 연결합니다.]]
4. [[guide/google-play-games-login-integration/4-unity-client-google-play-games-login/4-unity-client-google-play-games-login|Unity 클라이언트에서 Google Play Games와 Unity Authentication 로그인을 구현합니다.]]

## 내부 테스트 진행

내부 테스트는 Play 스토어를 통해 서명된 AAB를 설치하게 하는 배포 절차입니다. Play Games Services 테스트 사용자 등록과는 별도이므로, 같은 Google 계정을 두 곳 모두에 등록해야 합니다.

시작하기 전에 다음을 준비합니다.

- 이전에 배포한 빌드보다 큰 `versionCode`를 가진 서명된 AAB
- 내부 테스트와 Play Games Services 테스트에 사용할 Google 계정

### 1. 내부 테스트 트랙 열기

Play Console에서 **Test and release > Testing > Internal testing**을 엽니다. 처음 설정하는 앱에서는 `Select testers`, `Create a new release`, `Preview and confirm the release` 순으로 완료할 작업이 표시됩니다.

![[5-google-play-games-internal-testing-001.png]]

### 2. 테스터 선택

**Testers** 탭에서 이메일 목록을 새로 만들거나 기존 목록을 선택한 뒤 저장합니다. 이 목록에 포함된 계정만 내부 테스트 참여 링크를 통해 Play 스토어 배포본을 설치할 수 있습니다.

![[5-google-play-games-internal-testing-002.png]]

> [!NOTE]
> 이 목록은 Play 스토어에서 배포본을 설치할 계정을 관리합니다. Google Play Games 로그인까지 검증하려면 같은 계정을 [[#6. Play Games Services 테스트 사용자 등록|Play Games Services 테스트 사용자]]로도 추가하세요.

### 3. 내부 테스트용 AAB 업로드

**Releases** 탭에서 **Create new release**를 선택합니다. `Upload`로 새 AAB를 올리거나, 이미 업로드한 AAB가 있으면 `Add from library`에서 선택합니다.

![[5-google-play-games-internal-testing-003.png]]

AAB가 추가되면 버전 코드와 버전 이름을 확인하고 **Next**를 선택합니다.

![[5-google-play-games-internal-testing-004.png]]

### 4. 검토 후 게시

검토 화면에서 오류가 없는지 확인한 뒤 **Save and publish**를 선택합니다. `Missing debug symbols`와 같은 경고는 이 예시에서는 내부 테스트 게시를 막지 않지만, 경고 내용을 확인한 뒤 진행합니다.

![[5-google-play-games-internal-testing-005.png]]

게시 확인 대화상자에서 다시 **Save and publish**를 선택합니다. 이 변경 사항은 즉시 게시되며 Play 스토어에 반영되는 데 시간이 걸릴 수 있습니다.

![[5-google-play-games-internal-testing-006.png]]

### 5. 트랙 재개 및 참여 링크 확인

게시가 완료되어도 트랙이 일시중지 상태이면 테스터는 릴리스를 받을 수 없습니다. **Releases** 탭에 `This track is paused`가 보이면 **Resume track**을 선택해 트랙을 활성화합니다.

![[5-google-play-games-internal-testing-007.png]]

트랙을 재개한 뒤 **Testers** 탭에서 **Copy link**로 참여 링크를 복사합니다. 테스터 계정으로 이 링크를 열어 내부 테스트에 참여한 다음, Play 스토어에서 앱을 설치하거나 업데이트합니다.

![[5-google-play-games-internal-testing-008.png]]

### 6. Play Games Services 테스트 사용자 등록

Play Console의 **Play Games Services** 테스트 사용자 화면에서 내부 테스트에 사용한 것과 같은 Google 계정을 등록합니다. 내부 테스트 트랙 등록만으로는 Play Games Services 로그인 테스트 권한이 부여되지 않습니다.

![[5-google-play-games-internal-testing-009.png]]

왼쪽 메뉴에서 **Grow users > Play Games Services > Setup and management > Testers**로 이동한 뒤 **Add testers**를 선택합니다. 내부 테스트에 사용한 Google 계정을 추가합니다.

### 7. 실기기에서 로그인 확인

테스터 계정으로 Play 스토어 배포본을 설치한 Android 기기에서 앱을 실행합니다. Google Play Games 로그인과 Unity Authentication 로그인이 모두 성공하는지 확인합니다.

> [!TODO] 스크린샷 추가
> 내부 테스트 참여 완료 또는 Play 스토어 설치 화면과 앱의 로그인 성공 결과를 추가합니다.

## 참고자료

- [Test Google Play Games Services](https://developer.android.com/games/pgs/test?hl=ko)
