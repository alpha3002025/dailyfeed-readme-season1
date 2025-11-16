# member 서비스 설명

# Intro
JWT 기반의 인증을 사용합니다. AccessToken → RefreshToken → 로그아웃 시에는 Blacklist 등록 절차를 가집니다.

**OAuth2 는 v1 에서는 배제**<br/>
일반적으로는 OAuth2 를 사용하는 경우가 많지만 구글 심사 등의 과정이 꽤 오래걸리게 되고, local/dev 프로필에서만 실행해야 할 경우 개발을 위한 구글 인증 시크릿을 공유해야 하는데, 백엔드의 경우 Github Secrets 를 통해 공유할 수 있겠지만, Frontend 의 경우 직접 파일을 전달하거나 하는 과정이 필요할 수 있어서 이번 개발 주기인 season1 개발에서는 OAuth2 기반의 인증은 배제했습니다.<br/>

code 를 발급받은 이후부터 다중화를 지원하는 백엔드에 code 를 보관하는 방식 역시 지금 당장에는 복잡해질 수 있기에 이번 개발 주기에는 보류했습니다. Season2 에서 개발 예정입니다.
