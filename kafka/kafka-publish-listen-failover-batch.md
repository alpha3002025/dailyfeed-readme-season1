# Kafka - publish/listen 시의 에러처리, Failover 방식

# 요약
Publish/Listen 시의 에러 처리와 Fail over 를 위한 배치는 다음과 같이 수행합니다. 

Publish/Listen 공통

- message 전송시에는 항상 `messageKey` 라고 하는 고유하게 인식 가능한 일련번호를 생성합니다. 형식은 아래에서 자세히 설명합니다.
- publish, listen 시에는 중복 수신한 데이터인지를 체크합니다. 중복체크를 할때 redis 를 활용합니다.

- 그 외에도 중복수신을 하더라도 mongodb 의 upsert를 통해 데이터 저장시 에러를 보완합니다.

Publish

- 전송 실패시 `kafka_publish_dead_letters`  라는 mongodb 컬렉션에 저장
- deadletter 역시 저장 실패시 트랜잭션을 실패시킴 (트랜잭션 거부)

Listen

- listener 에서 `member_activities` 에 저장 실패시 `kafka_listener_dead_letters`  라는 mongodb 컬렉션에 저장
-  `kafka_listener_dead_letters`  라는 mongodb 컬렉션에 저장실패시 pvc 기반의 logger 로 해당 이벤트 내의 postId, memberId, commentId, activityEventType 을 기록하도록 지정

batch 

- publish 보정 
  - read: `kafka_publisher_dead_letters` 에서 `is_completed = false` 인 데이터를 N 건 읽어들인 후
  - process:  MemberActivityDocument 로 변환 후
  - write:`member_activities` 에 저장을 수행하고, 저장을 마친 `kafka_publisher_dead_letters` 도큐먼트는 `is_completed = true` 로 마스킹
- listener 보정
  - read : `kafka_listener_dead_letters` 에서 `is_completed = false` 인 데이터를 N 건 읽어들인 후
  - process : MemberActivityDocument 로 변환 후
  - write: `member_activities` 에 저장을 수행하고, 저장을 마친 `kafka_listener_dead_letters` 도큐먼트는 `is_completed = true` 로 마스킹
- pvc 로그 분해
  - 죄송합니다. 취업준비 때문에 바빠서 이것 까지는 못하겠습니다 하하하. (저 코딩테스트도 준비하고 이력서도 준비해야해요.)

<br/>

# Message Key 형식


# Publish


# Listener


# Batch
