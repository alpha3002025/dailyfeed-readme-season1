# member 서비스 설명
> claude 의 힘을 빌리지 않은 지극히 주관적인 설명 문서입니다.


dailyfeed-member 는 JWT 기반의 인증을 사용합니다. AccessToken → RefreshToken → 로그아웃 시에는 Blacklist 등록 절차를 가집니다.<br/>
<br/>


# 목차
- [Readme](./Readme.md)
- [0\. JWT 관리](./0.%20JWT%20관리.md)
- [1\. 인증 흐름](./1.%20인증%20흐름.md)
- [2\. logout 그리고 Blacklist](./2.%20logout%20그리고%20Blacklist.md)
- [3\. 서비스간 통신 시 인증 유효성 체크](./3.%20서비스간%20통신%20시%20인증%20유효성%20체크.md)
- [3\. ArgumentResolver 기반 인증 유효성 체크](./3.%20ArgumentResolver.md)

<br/>


# AccessToken,RefreshToken,Blacklist 방식의 인증 방식 채택
현재 프로젝트에서는 AccessToken, RefreshToken, Blacklist 방시의 인증방식을 선택해서 개발했습니다.<br/>

문서를 읽어보시기 전에 AccessToken, RefreshToken, Blacklist 방식에 대해 사전지식이 없다면 다음 내용을 확인해주시기 바랍니다.<br/>

| 요소                     | 역할                   | 특징                                                                                                                                          |
| ---------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Access Token (JWT)** | **API 접근 인증**        | - 유효 기간이 **짧음** (예: 15분, 1시간) - 필요한 사용자 정보(ID, 역할 등)를 담고 있음 - 서버는 요청마다 이 토큰의 유효성만 검증 (DB 조회 불필요) - 탈취되어도 짧은 시간 안에 만료되어 피해 최소화               |
| **Refresh Token**      | **Access Token 재발급** | - 유효 기간이 **긺** (예: 7일, 30일) - Access Token이 만료되었을 때, 새로운 Access Token을 발급받기 위해 사용 - 그 자체로 API 접근 권한은 없음 - 서버의 특정 저장소(DB, Redis 등)에 저장되고 관리됨 |
| **Blacklist**          | **로그아웃/탈취된 토큰 무효화**  | - 로그아웃 등으로 만료되지 않았지만 더 이상 사용하면 안 되는 토큰들을 저장하는 목록 - 주로 Redis와 같은 빠른 In-Memory DB에 저장 - Access Token이 여전히 유효하더라도 Blacklist에 있으면 접근을 거부        |
<br/>


## 전체 인증 흐름
#### 단계 1: 최초 로그인 (Login)
<br/>

![](./img/authentication/thumbnail-login.png)
<br/>

1. **사용자**가 아이디/비밀번호로 로그인을 요청합니다.
2. **서버**는 사용자 정보를 확인하고 인증에 성공하면:
    - **Access Token** (짧은 유효기간)을 생성합니다.
    - **Refresh Token** (긴 유효기간)을 생성합니다.
3. **서버**는 Refresh Token을 자체 DB나 Redis에 저장하고, 두 토큰을 모두 **사용자**에게 전달합니다.
4. **사용자(클라이언트)**는 두 토큰을 안전한 곳(보통 Access Token은 메모리, Refresh Token은 HttpOnly 쿠키)에 저장합니다.

<br/>

#### 단계 2: API 요청 (API Request)
<br/>

1. **사용자**는 API를 호출할 때마다 `Authorization` 헤더에 **Access Token**을 담아 보냅니다.
2. **서버**는 다음을 검증합니다.    
    - **Blacklist 확인**: 해당 Access Token이 Blacklist에 등록되어 있는지 확인합니다. 등록되어 있다면, 즉시 요청을 거부(401 Unauthorized)합니다.
    - **서명 및 유효기간 확인**: 토큰의 서명이 올바른지, 그리고 만료되지 않았는지 확인합니다.
3. 모든 검증을 통과하면, 서버는 요청을 처리하고 결과를 반환합니다.

> 2 의 단계에 대해 Blacklist 조회시 잦은 DB 접근을 할 경우 성능저하의 우려가 있기에 Redis 를 통해 캐싱을 해두었습니다.

<br/>

#### 단계 3: Access Token 만료 시 재발급 (Token Refresh)
<br/>

1. 사용자의 **Access Token**이 만료되면, API 요청은 실패(401 Unauthorized)합니다.
2. **사용자(클라이언트)**는 이 응답을 받으면, 가지고 있던 **Refresh Token**을 `/refresh` 와 같은 특정 API 엔드포인트로 보냅니다.
3. **서버**는 받은 Refresh Token을 DB에 저장된 값과 비교하여 유효성을 검증합니다.
4. 검증이 성공하면, 새로운 **Access Token**을 발급하여 사용자에게 전달합니다. (보안 강화를 위해 이 시점에 새로운 Refresh Token을 함께 발급하고 기존 것은 무효화하는 'Refresh Token Rotation' 전략도 많이 사용됩니다.)
5. **사용자**는 새로 받은 Access Token으로 이전에 실패했던 API 요청을 다시 시도합니다.

<br/>

![](./img/authentication/thumbnail-refresh-when-accesstoken-expired.png)

<br/>

#### 단계 4: 로그아웃 (Logout)
<br/>

1. **사용자**가 로그아웃을 요청합니다.    
2. **서버**는 요청 헤더에 담긴 **Access Token**을 가져와 **Blacklist**에 추가합니다. 이때, 토큰의 남은 유효기간만큼만 Blacklist에 저장하여(TTL 설정) 불필요한 데이터가 쌓이는 것을 방지합니다.
3. 동시에 서버 DB에 저장된 사용자의 **Refresh Token**을 삭제하거나 비활성화합니다.
4. 이렇게 하면, 해당 사용자는 Access Token이 만료 전이라도 더 이상 API를 사용할 수 없고, 새로운 Access Token도 발급받을 수 없게 됩니다.

![](./img/authentication/logout.png)

<br/>

#### refreshToken 만료시
<br/>

e.g. 로그인한 후 오랫동안 미접속



![](./img/authentication/thumbnail-relogin-when-refreshtoken-expired.png)


<br/>


refreshToken 이 만료되는 경우는 보통 오랫동안 로그인하지 않았을 경우입니다. 현재 프로젝트 내에서는 refreshToken 의 만료기한을 30Day 로 설정했기 때문에, 로그인한 날짜로부터 30일 이상 경과할 경우 refreshToken 의 기한이 만료되었으므로 재로그인을 하도록 `401 UnAuthorized {X-Relogin-Required}` 를 응답 상태코드/헤더로 응답합니다.<br/>
<br/>


# Notice
## OAuth2 는 Season 1 에서는 배제
OAuth 기반의 인증 (OAuth2.0, 구글로그인) 을 도입하려 했지만 다음의 이유들로 인해 배제하기로 했습니다.
- 구글 심사 등의 과정이 오래걸린다는 점 : 시간 상 3개월 안에 다른 프로젝트의 구현까지 완료하고 문서까지 작성해야 했기 때문에 개발 기한상 불가능하다고 판단
- 구글 인증 관련 local/dev 개발환경 구축 후 공개시 API 키/시크릿 공유를 해야할 수도 있다는 점
	- 이 부분에 대해서는 Season2 에서 다양한 방법을 도입해둘 예정입니다.

