> obsidian (https://obsidian.md) 를 통해 현재 리포지터리 디렉터리를 열어서 확인하시면 조금 더 가독성 높게 내용을 파악하실수 있습니다.

<br/>

# Dailyfeed 소개

> 참고) ~dev 프로필 사용시 사용 가능 시간~
> - ~비용 이슈로 인해 매일 9:00 \~ 18:00 까지만 RDS 를 켜두고 있습니다. Atlas MongoDB 역시 9:00 이후에 Network Access 를 공개하기에 dev 프로필의 경우 9:00 이후로 확인이 가능합니다.~
> - (2025.12.11) 비용 이슈로 인해 dev 프로필은 현재 운영하지 않습니다. 비용이슈로 RDS를 제거하게 되면서 dev 프로필도 더 이상 관리하지 않기로 했습니다.

<br/>

# 프로젝트 설명
- 소개: 사용자가 회원가입 후 로그인을 해서 다른 사용자를 팔로우를 하고, 팔로잉 중인 멤버의 글들을 확인하는 SNS 형태의 프로젝트입니다. k8s 환경에서 인스턴스를 어떻게 구분하는 것이 좋은지에 초점을 맞춰서 설계를 했었습니다.
- RDB + NoSQL: 정규화된 구조화 데이터와 검색에 최적화된 NoSQL 을 혼용하는 방식을 채택하고 있습니다. 일관성 있는 조회와 JOIN 에 최적화된 관계형 데이터베이스의 장점을 살리되 데이터의 검색에는 NoSQL 의 힘을 빌리도록 역할을 분담한 설계입니다.
- Kafka: Kafka 가 엄청 필요한 주제는 아니었지만 카프카 개념 설명과 설계능력, 에러 대응 등에 대해 설명을 위해 Kafka 를 사용하게 되었습니다.
  - 개인적으로는 카프카의 통신 인프라는 가벼워야 하며 inbox, outbox 에 수신/송신한 이벤트 메시지를 기록하는 것 까지가 최소 구현이며 그 뒤에서부터는 비즈니스 워크로드를 풀어내기 위한 스케일링 그룹의 역할이라고 생각합니다. Kafka 통신인프라에 비즈니스 워크로드 까지 처리하면서 비즈니스 워크로드가 먹통이 되면 Kafka 통신까지 두절되는 것은 치명적이라고 생각해서입니다.
  - Kafka 의 신뢰성있는 통신을 보장하며 통신 기록을 지속(Persist)할수 있고, 다른 메시지큐와는 다르게 컨슈머가 구독을 하는 방식으로 하는 장점에 대해서는 알고 있긴 하지만, 개인적으로는 Kafka 가 비즈니스까지 처리하는 것은 통신을 무결하게 할지 비즈니스를 무결하게 처리할지 설계의 혼돈지점에 있다고 생각합니다. 하지만, 많은 회사의 채용공고를 보면 Kafka 를 스케일아웃을 위한 비즈니스 워크로드 클러스터로 쓰는 것 같은 느낌의 공고를 많이 보긴 한것 같습니다.
- (참고)
  - 현재 프로젝트는 2025년도에 만든 프로젝트로, **'프로젝트의 단점과 개선 방향 (2026.08)'** 섹션에서 언급하는 방향으로 설계의 관점을 변경 예정입니다.
  - “kubernetes 를 이용해 워크로드를 성격별로 분리하고 스케일아웃을 용도별로 할 수 있도록 하는 것”이 처음 목표였지만, MSA 및 분산 아키텍쳐구조에서 신뢰성 있는 이벤트 처리에 관련된 내용들을 AI 등을 이용해 학습하면서 설계 관점이 바뀌었습니다. 해당 내용에 대해 개선 프로젝트를 할 경우 어떤 방식의 접근을 할지에 대해 **'프로젝트의 단점과 개선 방향 (2026.08)'** 섹션에 정리해두었습니다.

# 프로젝트의 단점과 개선방향 (2026.08)
“kubernetes 를 이용해 워크로드를 성격별로 분리하고 스케일아웃을 용도별로 할 수 있도록 하는 것”이 초창기 목표였다면, 
현재(2026/08) 시점에서 마이크로서비스 패턴과 분산 아키텍쳐 구조에서 정합성을 확보하는 방식들에 대해 AI 등을 통해 학습을 해보다보니, 
몇몇 문제점을 발견했고, 제가 실수한 부분들에 대해 언급해보고자 합니다.

**(kafka) `@Transactional` 내에서 kafka publish 를 하는 실수를 범했다는 점, KafkaListener 내에서 @Transactional  한 작업이 수행된다는 점**<br/>
publish 를 @Transactional 에서 수행하는 코드가 있습니다. 또한 KafkaListener 내에서 @Transactional 을 수행 중입니다. 이 부분에 대해서는 다음과 같이 개선하면 좋을 것 같고, 추후 프로젝트 진행시 고려해보려 합니다.
- publish: outbox 에 이벤트(메시징) 데이터를 저장, 비동기적으로 외부의 스케쥴링된 작업에서 이를 주기적으로 대사 처리
- listener: inbox 에 이벤트(메시징) 데이터를 저장, 비동기적으로 데이터를 소비
- 메시지의 순서가 균일하게 이뤄져야 하는 경우(e.g. 주문, 결제, 결제 승인, 포인트 적립, 배송업데이트 등) SAGA 패턴 기반의 파이프라인 구축
메시징 내에 @Transactional 이 아예 없을수는 없지만, 메시징 영역내에서는 여러 가지 트랜잭션 작업을 수행하기보다 inbox/outbox 트랜잭션에 국한되어 수행하도록 하겠습니다.

**(kafka) 서비스 API 호출이 @Transactional 내에서 수행된다는 점**<br/> 
MSA API 호출을 아예 배제할수는 없습니다. Transactional 영역 내에서 API 호출 실패할 경우, 실패판정 응답 등에 대해서 별도의 보상(예외처리) 후처리를 하도록 하는 로직을 거치도록 하겠습니다.
이미 FeignResponse 를 처리하는 부분들에 에러코드와 Exception 을 분류하고 try, catch 를 통해 해당 부분에 대해 일부 구현은 되어 있는데 더 확인해보고 새는 부분이 있는지 검토예정입니다.

**(inbox/outbox) inbox/outbox 테이블 설계**<br/>
inbox/outbox 테이블은 운영이 길어질 수록 비대해집니다. 따라서 주기적인 삭제가 필요하며 별도의 파티셔닝과, 가장 최근 요청을 읽어들이기 쉽도록 하는 index 조합이 중요합니다. inbox/outbox 구조로 전환시 여기에 대한 설정을 해둘 예정입니다.

**(Infra, Terraform) Infra 구성에는 Terraform, EKS, AWS CLI, SecretsManager, IRSA 적용, Role Based Access 를 활용**<br/>
이 프로젝트를 할 때는 CKA, ICA 를 취득했던 시점이며, Terraform 에 대해 잘 모르던 시점입니다. 현재는 Terraform 에 익숙해진 상황입니다. 따라서 프로젝트를 재설계를 하게 된다면 Infrastructure 의 경우 Terraform, Helm 을 통해 구성하고 주요 보안속성들은 Secrets Manager를 활용할 것 같고, Pod 의 Container 등에 대해 IRSA 를 적용하겠습니다. AWS CLI 를 사용하는 환경 역시 API Access Key/Secret 방식의 옛날의 방식 대신 Role Based 기반의 접근 방식을 통해 개발환경에 대한 보안도 신경쓰도록 하는 것이 나아보입니다.



# 프로젝트 설명
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
