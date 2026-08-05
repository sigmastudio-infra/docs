# 서버 토큰 발행+ 구현 가이드  
  
## 1. 목적  
  
이 문서는 서버에서 UGS 인증 토큰 검증 이후, 자체 Access Token / Refresh Token을 발행하는 과정을 설명합니다.  
  
클라이언트는 Unity Authentication에서 받은 UGS JWT를 서버로 전달하고, 서버는 해당 토큰을 검증한 뒤 자체 세션을 생성합니다.

## 2. 전체 로그인 흐름  
  
1. 클라이언트가 Google Play Games 로그인  
2. Unity Authentication에서 UGS JWT 발급  
3. 클라이언트가 서버에 UGS JWT 전달  
4. 서버가 UGS JWT 검증  
5. 서버가 유저 조회 또는 신규 생성  
6. 서버가 세션 생성  
7. 서버가 자체 Access Token / Refresh Token 발급  
8. 클라이언트는 이후 요청부터 서버 Access Token 사용

## 3. UGS JWT 검증  
  
서버는 클라이언트가 보낸 UGS JWT에 대해 다음 항목을 검증합니다.  
  
- RS256 서명 검증  
- `iss`가 Unity Authentication issuer와 일치하는지 확인  
- `aud`에 `upid:<projectId>`와 `envId:<envId>`가 포함되어 있는지 확인  
- `token_type = authentication` 확인  
- `exp`, `nbf`, `iat` 시간 클레임 검증  
- `sub`, `jti`, `sign_in_provider` 존재 여부 확인  
- `sign_in_provider`가 `anonymous`가 아닌지 확인  
  
검증이 통과되면 서버는 `sub`를 외부 인증 식별자로 사용합니다.

##  4. 유저 등록 / 조회 규칙  
  
UGS JWT의 `sub`는 Unity Authentication 기준의 유저 식별자입니다.  
  
서버는 이 값을 그대로 내부 `user_id`로 사용하지 않고, 자체 `user_id`를 발급합니다.

##  5. 서버 검증 설계 

현재 서버는 `ktor-server-auth-jwt` 기반으로  
UGS JWT를 검증합니다.  
  
Unity Authentication의 JWKS endpoint에서 공개키를 가져와  
RS256 서명을 검증하며, 검증 성공 시 JWT payload를 기반으로 로그인 처리를 진행합니다.

--- 
 
### 5-1. Audience 검증  
  
서버는 `aud` 배열 내부에 다음 값이 포함되어 있는지 확인합니다.  
  
```text  
upid:<UGS Project Id>  
envId:<UGS Environment Id>  
```  
  
이 검증을 통해 다른 Unity 프로젝트에서 발급된 JWT 사용을 방지합니다.  
  
---  
  
### 5-2. token_type 검증  
  
UGS JWT는 여러 목적의 토큰이 존재할 수 있기 때문에, 서버는 반드시 다음 값을 검증합니다.  
  
```text  
token_type = authentication  
```  
  
이를 통해 로그인용 JWT만 허용합니다.  
  
---  
  
### 5-3. Leeway 사용  
  
모바일 환경에서는 클라이언트와 서버 간 시간 차이가 발생할 수 있습니다.  
  
이를 고려하여:  
  
```kotlin  
acceptLeeway(60)  
```  
  
설정을 사용합니다.  
  
현재 서버는 최대 60초의 clock skew를 허용합니다.  
  
---  
  
### 5-4. sign_in_provider 검증  
  
현재 서버는 anonymous 로그인 사용을 허용하지 않습니다.  
  
따라서:  
  
```text  
sign_in_provider != anonymous  
```  
  
조건을 검증합니다.  
  
현재 허용 provider:  
  
- google-play-games 

## 6. Replay 공격 방지  
  
UGS JWT는 탈취 가능성을 고려해야 합니다.  
  
동일한 JWT가 재사용되는 것을 방지하기 위해,  서버는 JWT의 `jti`를 저장합니다.  

  ---
  
### 6-1. consumed_ugs_jti 테이블  
  
```sql  
CREATE TABLE consumed_ugs_jti (  
jti TEXT PRIMARY KEY,  
sub TEXT NOT NULL,  
expires_at TIMESTAMP NOT NULL  
)  
```  
  
---  
  
### 6-2. 동작 방식  
  
로그인 성공 전에 다음 작업을 수행합니다.  
  
```sql  
INSERT INTO consumed_ugs_jti (  
jti,  
sub,  
expires_at  
)  
VALUES (...)  
```  
  
이미 동일한 `jti`가 존재하면  
중복 로그인 요청으로 판단하고 거부합니다.  
  
이를 통해 JWT replay 공격을 방지합니다.
