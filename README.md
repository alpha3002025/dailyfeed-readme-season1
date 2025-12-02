> obsidian (https://obsidian.md) 를 통해 현재 리포지터리 디렉터리를 열어서 확인하시면 조금 더 가독성 높게 내용을 파악하실수 있습니다.

<br/>

# Dailyfeed 소개

> 참고) dev 프로필 사용시 사용 가능 시간
> - 비용 이슈로 인해 매일 9:00 \~ 18:00 까지만 RDS 를 켜두고 있습니다. Atlas MongoDB 역시 9:00 이후에 Network Access 를 공개하기에 dev 프로필의 경우 9:00 이후로 확인이 가능합니다.

<br/>


# 프로젝트 설명
사용자가 회원가입 후 로그인을 해서 다른 사용자를 팔로우를 하고, 팔로잉 중인 멤버의 글들을 확인하는 SNS 형태의 프로젝트입니다. k8s 환경에서 인스턴스를 어떻게 구분하는 것이 좋은지에 초점을 맞춰서 설계를 했습니다.
<br/>

# 프로젝트 문서
pdf 로 정리한 문서는 다음과 같습니다.<br/>
- [dailyfeed 프로젝트 설명문서.pdf (다운로드 페이지로 이동)](./dailyfeed-프로젝트-설명문서-2025.pdf) 
- https://github.com/alpha3002025/dailyfeed-readme-season1/blob/main/dailyfeed-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%A4%EB%AA%85%EB%AC%B8%EC%84%9C-2025.pdf
<br/>
<br/>

만약 조금 더 상세하고 TMI 에 가까운 설명문서를 확인하고 싶으시다면 다음 링크를 확인해주세요.
- [tmi-docs/Readme.md](./tmi-docs/Readme.md)

<br/>
<br/>

# 주요 기능
## 로그인 페이지

로그인 페이지입니다. 이메일 회원가입이 가능합니다.

![](./img/readme/documentation/login.png)
<br/>
<br/>

## My follow's news

내가 팔로우 하고 있는 멤버들이 작성한 글들을 최근 작성 순으로 확인할 수 있습니다.

![](./img/readme/documentation/1-my-follows-news.png)
<br/>



## Most popular now

가장 인기있는 글들을 확인할 수 있습니다. (조회수 + 좋아요 x 2)

![](./img/readme/documentation/2-most-popular-now.png)
<br/>



## Most comments now

댓글 많이 달린 글들을 조회합니다.

![](./img/readme/documentation/3-most-comments-now.png)
<br/>



## My feed

내가 작성한 글들을 확인합니다.

![](./img/readme/documentation/4-my-feed.png)
<br/>

## Post 상세

글 상세 화면입니다.

![](./img/readme/documentation/7-post-detail.png)
<br/>

## Profile

프로필 페이지입니다. 프로필을 수정하거나 썸네일을 수정할 수 있습니다.

![](./img/readme/documentation/5-profile.png)
<br/>

## Connections

follow, following 목록을 확인할 수 있는 페이지입니다.

![](./img/readme/documentation/6-connections-1.png)
<br/>

![](./img/readme/documentation/6-connections-2.png)
<br/>


<br/>



# github
**설치**
> infra 설치, helm 차트

- Project: https://github.com/alpha3002025/dailyfeed-installer
  - module: https://github.com/alpha3002025/dailyfeed-infrastructure : kind,mysql,kafka,redis,configmap 등을 설치 및 관리
  - module: https://github.com/alpha3002025/dailyfeed-app-helm : 애플리케이션의 helm 및 설치 스크립트를 관리
<br/>

Frontend
> Next.js App Router 기반의 애플리케이션이며, Frontend 학습을 위해 2주 이상을 학습했었지만, 막상 프로젝트를 하다보니 시간관리를 위해 95% 이상의 코드를 AI(Claude Code)를 이용해 개발하게 되었습니다.<br/>

- Project: https://github.com/alpha3002025/dailyfeed-frontend-svc
<br/>

계정 서비스
- Project: https://github.com/alpha3002025/dailyfeed-member-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-member
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
<br/>

콘텐츠 서비스
- Project: https://github.com/alpha3002025/dailyfeed-content-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-content
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
  - module: https://github.com/alpha3002025/dailyfeed-kafka-support
<br/>

timeline 서비스
> 피드, 인기있는글들, 댓글많은 글, 댓글수 카운팅 등, 조회에 관련된 기능을 담당)

- Project: https://github.com/alpha3002025/dailyfeed-timeline-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-timeline
  - module: https://github.com/alpha3002025/dailyfeed-pvc-support
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
  - module: https://github.com/alpha3002025/dailyfeed-kafka-support
<br/>

image 서비스
> 서비스 (e.g. 썸네일)

- Project: https://github.com/alpha3002025/dailyfeed-image-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-image
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
<br/>

검색 서비스
> (e.g. 본문검색, Full Text Search)

- Project: https://github.com/alpha3002025/dailyfeed-search-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-search
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
<br/>

멤버 활동 기록 서비스
- Project: https://github.com/alpha3002025/dailyfeed-activity-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-activity
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-pvc-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
  - module: https://github.com/alpha3002025/dailyfeed-kafka-support
<br/>

배치 서비스
- Project: https://github.com/alpha3002025/dailyfeed-batch-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-batch
  - module: https://github.com/alpha3002025/dailyfeed-pvc-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support

<br/>
