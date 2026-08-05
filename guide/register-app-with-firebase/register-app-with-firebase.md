---
references: https://firebase.google.com/docs/unity/setup#register-app
---
# Register your app with Firebase

## 앱 추가하기

![[register-app-with-firebase-001.png]]
**Add app** 을 누르면 다음과 같이 플랫폼을 선택할 수 있음

![[register-app-with-firebase-002.png]]

원하는 플랫폼을 선택하자

## Unity app Firebase에 등록하기

![[register-app-with-firebase-003.png]]
Unity 게임을 iOS와 Android 양쪽으로 출시한다면, **두 플랫폼을 각각 별도의 Firebase 프로젝트로 만들지 말고, 하나의 동일한 Firebase 프로젝트 안에 둘 다 등록**해야 함

> [!NOTE] 
> 
> 참고로 iOS 플랫폼의 **bundle ID** 와 Android**packagename**은 동일하게 맞추는게 좋음

## config file 다운로드 하기

![[register-app-with-firebase-004.png]]
`google-services.json` 이라는 설정 파일을 다운로드 받고, Unity project의 `Assets` 아래에 두면 된다.

## Firebase Unity SDK 다운로드

![[register-app-with-firebase-005.png]]

필요한 SDK를 다운로드 받고, **Assets > Import Package > Custom Package** 메뉴에서 추가
