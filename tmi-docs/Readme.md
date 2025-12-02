> 현재 리포지터리내의 모든 문서는 obsidian 에서 열어서 보시면 깔끔하게 보입니다.

<br/>

'dailyfeed' 는 취업을 준비하면서 CKA,ICA 자격증을 취득했지만 단순히 자격증만으로는 뭔가를 증명하기에는 아쉽다고 시작해서 시작한 프로젝트입니다. "프로젝트가 점점 복잡해지는 것일까?" 하는 죄책감도 가끔 들었는데, 막상 또 따지고 보면 단순하기도 합니다. (로컬에서 모든 인프라를 구비하다보니 인프라 설치가 복잡해서 복잡해졌고, 인증 쪽이 복잡해서 복잡해보이는데 막상 content, timeline 코드는 단순합니다.) <br/>

현재 디렉터리 (tmi-docs) 부터는 TMI 버전의 프로젝트 설명 문서입니다. 설명문서 pdf 를 보신 분들도 있겠지만, TMI버전을 더 좋아하시는 분들을 위해 가급적 깔끔한 버전의 프로젝트 설명문서를 준비했습니다.<br/>
<br/>


# 목차
[member/](./member)
- [Readme](./member/Readme.md)
- [0\. JWT 관리](./member/0.%20JWT%20관리.md)
- [1\. 인증 흐름](./member/1.%20인증%20흐름.md)
- [2\. logout 그리고 Blacklist](./member/2.%20logout%20그리고%20Blacklist.md)
- [3\. 서비스간 통신 시 인증 유효성 체크](./member/3.%20서비스간%20통신%20시%20인증%20유효성%20체크.md)
- [3\. ArgumentResolver 기반 인증 유효성 체크](./member/3.%20ArgumentResolver.md)
<br/>

[system-design/](./system-design/)
- [Readme](./system-design/Readme.md)
- [0\. 도메인 경계 정의 (스케일아웃 성격별 분류)](./system-design/0.%20도메인%20경계%20정의%20(스케일아웃%20성격별%20분류).md)
- [1\. 전체 구성도](./system-design/1.%20전체%20구성도.md)
<br/>

[kafka/](./kafka/)
- [Readme](./kafka/Readme.md)
- [0.introduce-problem](./kafka/0.introduce-problem.md)
- [1.process-kafka-failure](./kafka/1.process-kafka-failure.md)
- [2.producer-acks-설정과-consumer-offset-commit-설정](./kafka/2.producer-acks-설정과-consumer-offset-commit-설정.md)
- [3.중복-메시지-체크-방식](./kafka/3.중복-메시지-체크-방식.md)
- [4.날짜-별-토픽-운영](./kafka/4.날짜-별-토픽-운영.md)
- [5.publish-type-으로-kafka,feign-통신방식-선택](./kafka/5.publish-type-으로-kafka,feign-통신방식-선택.md)
- [6.`dailyfeed-kafka-support` 모듈 및 kafka 설정](./kafka/6.kafka-support-모듈.md)
<br/>

[project-review/](./project-review/)
- [Readme](./project-review/Readme.md)
- [Supported-features](./project-review/Supported-features.md)
- [Not-Supported-features](./project-review/Not-Supported-features.md)

<br/>
<br/>

> 현재 프로젝트 명은 'dailyfeed' 이고 '일상공유피드' 개념이지만, season2 에서는 'dailyfeed'라는 이름을 바꿔서 스포츠 관련 피드 시스템을 만들어볼 예정입니다.

<br/>
