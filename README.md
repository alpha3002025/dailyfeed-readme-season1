> obsidian (https://obsidian.md) 를 통해 현재 리포지터리 디렉터리를 열어서 확인하시면 조금 더 가독성 높게 내용을 파악하실수 있습니다.

<br/>

# Dailyfeed 소개

> 참고) ~dev 프로필 사용시 사용 가능 시간~
> - ~비용 이슈로 인해 매일 9:00 \~ 18:00 까지만 RDS 를 켜두고 있습니다. Atlas MongoDB 역시 9:00 이후에 Network Access 를 공개하기에 dev 프로필의 경우 9:00 이후로 확인이 가능합니다.~
> - (2025.12.11) 비용 이슈로 인해 dev 프로필은 현재 운영하지 않습니다. 비용이슈로 RDS를 제거하게 되면서 dev 프로필도 더 이상 관리하지 않기로 했습니다.

<br/>


# 프로젝트 설명
> (2026.08 현재) Terraform, EKS, Secrets Manager 등에 어느 정도 익숙해진 현재 시점에서는 프로젝트를 다시 한다면... 
- 예제를 조금 더 다듬어서 교재로도 사용할수 있는 Terraform,EKS 기반의 프로젝트를 수행할것 같습니다.
- Kafka 의 경우는 범위에서 배제하고, 별도의 사이드 프로젝트로 분리해서 시작할 것 같습니다.
- `@Transactional` 안에서 수행되는 kafka publish/consumer 코드가 있는데, 아마도 다시 한다면 inbox/outbox 기반의 구조로 전환하고 `@Transactional` 내에서 카프카 로직이 수행되지 않도록 할 것입니다. 이 부분은 저의 실수였다고 생각합니다. `@Transactional` 과 Kafka 의 영역이 분리되어야 Kafka Offset 커밋이 누락되거나 잘못 커밋되는 증상에 대한 예외상황을 방지할수 있는데 이 부분을 놓쳤습니다.


> `~ 2025.12`
- 소개 : 사용자가 회원가입 후 로그인을 해서 다른 사용자를 팔로우를 하고, 팔로잉 중인 멤버의 글들을 확인하는 SNS 형태의 프로젝트입니다. k8s 환경에서 인스턴스를 어떻게 구분하는 것이 좋은지에 초점을 맞춰서 설계를 했었습니다.<br/>
- RDB + NoSQL : 정규화된 구조화 데이터와 검색에 최적화된 NoSQL 을 혼용하는 방식을 채택하고 있습니다. 일관성 있는 조회와 JOIN 에 최적화된 관계형 데이터베이스의 장점을 살리되 데이터의 검색에는 NoSQL 의 힘을 빌리도록 역할을 분담한 설계입니다.
- Kafka : Kafka 가 엄청 필요한 주제는 아니었지만 카프카 개념 설명과 설계능력, 에러 대응 등에 대해 설명을 위해 Kafka 를 사용하게 되었습니다.
  - 개인적으로는 카프카의 통신 인프라는 가벼워야 하며 inbox, outbox 에 수신/송신한 이벤트 메시지를 기록하는 것 까지가 최소 구현이며 그 뒤에서부터는 비즈니스 워크로드를 풀어내기 위한 스케일링 그룹의 역할이라고 생각합니다. Kafka 통신인프라에 비즈니스 워크로드 까지 처리하면서 비즈니스 워크로드가 먹통이 되면 Kafka 통신까지 두절되는 것은 치명적이라고 생각해서입니다.
  - Kafka 의 신뢰성있는 통신을 보장하며 통신 기록을 지속(Persist)할수 있고, 다른 메시지큐와는 다르게 컨슈머가 구독을 하는 방식으로 하는 장점에 대해서는 알고 있긴 하지만, 개인적으로는 Kafka 가 비즈니스까지 처리하는 것은 통신을 무결하게 할지 비즈니스를 무결하게 처리할지 설계의 혼돈지점에 있다고 생각합니다. 하지만, 많은 회사의 채용공고를 보면 Kafka 를 스케일아웃을 위한 비즈니스 워크로드 클러스터로 쓰는 것 같은 느낌의 공고를 많이 보긴 한것 같습니다.
<br/>


> `2026.04.01`
- HPA 최적화를 위해 Resource, Requests/Limit 을 맞춰나갔던 작업 역시 꽤 힘든 작업이었습니다. 실무에서 k8s 를 적용할 때 user-svc, post-svc 이런 식으로 배포했을 경우 새로 추가되는 기능이 어느 정도의 리소스를 잡아먹을지 판단이 쉽지 않아지고 잠재적인 장애를 유발할수 있다는 점 역시 알게되었던 것 같습니다. 작년 여름에 이 프로젝트를 끝내고, 취업준비를 위헤 코딩테스트 준비를 하는 동안 잠시 프로젝트를 접어뒀다가 다시 프로젝트를 돌아보니 HPA 적용시에 각 서비스별 리소스의 최소/최대치를 설계하는 작업 역시 물리적인 시간을 상당히 많이 잡아먹었던 힘들었던 작업이었다고 생각합니다. 그 당시에는 리소스 스펙을 체크하는 데에 이렇게 시간을 잡아먹고(1주일정도)나니 자괴감이 들었던 기억이 있었는데 사람 마음이 간사하다고 힘든 기억을 잊고 있다가 k8s 의 Scheduling 파트를 복습해보는 과정에서 떠올랐네요!!<br/>
<br/>


# 프로젝트 문서
pdf 로 정리한 문서는 다음과 같습니다.<br/>
- [dailyfeed 프로젝트 설명문서.pdf (다운로드 페이지로 이동)](./dailyfeed-project-설명문서.pdf) 
- https://github.com/alpha3002025/dailyfeed-readme-season1/blob/main/dailyfeed-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%84%A4%EB%AA%85%EB%AC%B8%EC%84%9C-2025.pdf
<br/>
<br/>

현재 문서는 새로운 버전으로 작성해두었습니다. 이 리포지터리는 외부에 공개하고 있지 않습니다.
- https://nextradocs-dailyfeed-season1.vercel.app/

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

**Frontend**
> Next.js App Router 기반의 애플리케이션이며, Frontend 학습을 위해 2주 이상을 학습했었지만, 막상 프로젝트를 하다보니 시간관리를 위해 95% 이상의 코드를 AI(Claude Code)를 이용해 개발하게 되었습니다.<br/>

- Project: https://github.com/alpha3002025/dailyfeed-frontend-svc
<br/>

**계정 서비스**
- Project: https://github.com/alpha3002025/dailyfeed-member-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-member
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
  <br/>

**콘텐츠 서비스**
- Project: https://github.com/alpha3002025/dailyfeed-content-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-content
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
  - module: https://github.com/alpha3002025/dailyfeed-kafka-support
  <br/>

**timeline 서비스**
> 피드, 인기있는글들, 댓글많은 글, 댓글수 카운팅 등, 조회에 관련된 기능을 담당

- Project: https://github.com/alpha3002025/dailyfeed-timeline-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-timeline
  - module: https://github.com/alpha3002025/dailyfeed-pvc-support
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
  - module: https://github.com/alpha3002025/dailyfeed-kafka-support
  <br/>

**image 서비스**
> 서비스 (e.g. 썸네일)

- Project: https://github.com/alpha3002025/dailyfeed-image-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-image
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  <br/>

**검색 서비스**
> (e.g. 본문검색, Full Text Search)

- Project: https://github.com/alpha3002025/dailyfeed-search-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-search
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  <br/>

**멤버 활동 기록 서비스**
- Project: https://github.com/alpha3002025/dailyfeed-activity-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-activity
  - module: https://github.com/alpha3002025/dailyfeed-feign-support
  - module: https://github.com/alpha3002025/dailyfeed-pvc-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support
  - module: https://github.com/alpha3002025/dailyfeed-kafka-support
  <br/>

**배치 서비스**
- Project: https://github.com/alpha3002025/dailyfeed-batch-svc
  - module: https://github.com/alpha3002025/dailyfeed-code
  - module: https://github.com/alpha3002025/dailyfeed-batch
  - module: https://github.com/alpha3002025/dailyfeed-pvc-support
  - module: https://github.com/alpha3002025/dailyfeed-redis-support

<br/>
