고객사 요청을 접수하는 `front-desk` API 라고 이름 붙이겠습니다.<br/>
올바른 요청이거나 `front-desk` 에 장애가 없을 경우 요청을 기록해야 합니다.<br/>
요청을 기록하는 수단으로는 MySQL, Kafka 를 선택할수 있는데, 이 중 Kafka 를 메인 옵션으로 선택하며, 장애 상황에는 MySQL 의 `kafka_publisher_dead_letters` 테이블에 발급실패 메시지를 기록합니다. Kafka 를 요청 기록 수단으로 선택한 이유는 Kafka 역시 메시지를 영속화할 수 있기에 후행하는 Listener 서비스를 따로 돌려서 보정작업을 수행할수도 있다는 장점 때문에 '통신 계층'에 대한 메시지의 '영속성'확보를 위해 Kafka 를 선택했습니다.

`front-desk` API<br/>
`front-desk` 가 하는 동작을 요약해보면 다음과 같습니다.
- (1) 고객사에서 보낸 메시지 발송요청이 적합한지 판단 후 부적합할 경우 400 BAD REQUEST 를 내보내며, 발송 형식 중 잘못된 부분에 대한 원인역시 응답으로 내보냅니다.
- ~~(2) 자사 메시지 직렬화/역직렬화 에러 500 Internal Server Error 과 함께 '기술팀에 연락주시기 바랍니다.' 메시지 추가 (내부시스템에서도 에러 기록) 한 응답을 내보냅니다.~~
- (3) 카프카 토픽 발송 실패시 `kafka_publisher_dead_letters` 에 요청 기록
  - 이때 `kafka_publisher_dead_letters` 내의 필드 중 `triggered_at` 필드에 어디에서 난 장애인지를 기록합니다. (e.g. `triggered_at = 'PUBLISH_FAILURE'`)
- (4) 카프카 대신 다른 방식의 메시지 전송을 선택해야 할 경우 (e.g. 카프카 유지보수, 다른 기능테스트) 
  - `kafka_publisher_dead_letters` 에 요청 기록 
  - 이때 `kafka_publisher_dead_letters` 내의 필드 중 `triggered_at` 필드에 별도의 값을 기록합니다. (e.g. `triggered_at = 'USING_BATCH'`)
- (5) 정상적으로 publsh 가 될 경우 kafka message queue 에 발송될 객체 메시지의 필드 중에는 `curr_status` 라는 필드가 있는데 `curr_status = KAFKA_PUBLISH_SUCCESS`필드가 포함되어 메시지가 발송됩니다. 
- (6) 요청을 정상적으로 받았지만 Kafka publish 대신 다른 수단을 선택했거나, Kafka publish 를 실패한 케이스의 경우 `START` 를 기록합니다.

<br/>

> !TODO - 위의 (1) ~ (6) 중 (5),(6) 에서 사용하는 필드명과 `kafka_publish_dead_letters` 테이블 내의 컬럼들은 모호한면이 있기 때문에 table 을 분리할지를 결정해서 2026.05.24 이후 부터 업데이트하겠습니다.<br/>


카프카를 도입할지, 카프카 없이 Only 배치 기반으로 비동기적인 처리를 할지를 고민할 수 있는데, 제 경우에는 카프카 없이도 Only 배치 기반으로 비동기 처리를 할수도 있다고는 생각합니다. 다만 요청이 너무 높고, 앞단의 API 에 EKS 등을 도입하는 것보다 Kafka 를 도입하는게 경제적으로 더 이득이라면, 앞단의 API 에 요청을 기록하도록 하는 것이 나아보입니다.<br/>
<br/>

가장 처음 생각해볼 수 있는 두가지 선택지는 다음과 같습니다.<br/>

(1) 카프카 없이 배치만으로 비동기 시스템
- 고객사 -> `front-desk` API (`kafka_publisher_dead_letters` 에 요청 기록) (eks 기반) -> cronjob + spring batch (in_box 테이블의 데이터를 분류 후 처리)

(2) 카프카를 사용할 경우
- 고객사 -> `front-desk` API -> kafka publish -> kafka listener(in_box 적재 및 cronjob + spring batch (in_box 테이블의 데이터를 분류 후 처리))
- kafka listener 의 설계는 뒤에 이어지는 별도의 문서에서 정리합니다.

다만, 앞에서 미리 정의했듯 1초에 3000건 까지만을 기본적인 허용치로 정의했고, 고객사가 300곳 이상으로 넘어갈 경우 in_box 를 관계형 Database 로 둘수 없는 규모가 됩니다. 따라서 (2) 카프카를 사용할 경우를 채택합니다.

<br/>

## kafka publisher
두가지 요소를 고려해봅니다.

(1) 요청에 대한 ID 발급
- `{고객사 ID}-{발급수단Code}-{정수타입시간(Nano Second 까지표기)}`
- 고객당 1초에 3000건까지 허용했기에 1ms 에 3건까지는 발생할수 있습니다. 따라서 ms 아래 단위인 ns 까지도 허용합니다.
(2) kafka 발송 실패
- 카프카에 항상 발송이 성공한다고 보장한다면 그것은 혼자만의 '정신승리'일수 있습니다. 카프카 발송이 실패하는 경우 역시 고려해야 합니다.
- 이 경우 발송 실패 건은 이번 프로젝트 문서에서 다뤘듯 `kafka-publisher_dead_letters` 라는 테이블에 저장합니다.

메시지 형식<br/>
발송하는 메시지에 포함되어야 하는 정보는 다음과 같습니다.
- 메시지 ID : `{고객사ID}-{발급수단Code}-{정수타입시간(Nano Second 까지표기)}`
- 고객사 ID : 고객사 ID (SSG, 유니클로 등에 대해 자사에 분류되어 있는 ID)
- 발급수단 Code : 발급수단 Code (KAKAO,SMS)
- payload : 메시지 본문 (String)
- curr_status: START, KAFKA_PUBLISH_SUCCESS, KAFKA_PUBLISH_FAIL, IN_BOX_STAGING, IN_BOX_WAIT_KAKAO, IN_BOX_WAIT_SMS, SEND_SUCCESS
  - curr_status 는 Enum 형태의 상수코드입니다.
  - START : 요청을 받았고, PUBLISH_SUCCESS 의 전단계이며, Kafka 장애가 발생하는 경우와 Kafka publish 를 실패할 경우도 있기에 START 과정에 대한 코드도 두었습니다. 'START' 코드는 `kafka_publisher_dead_letters` 에만 기록되는 상수코드입니다.
  - KAFKA_PUBLISH_SUCCESS: kafka 토픽에 발송이 성공된 상태입니다.
  - KAFKA_PUBLISH_FAIL: kafka 토픽에 발송이 실패한 상태입니다. `kafka_publisher_dead_letters` 에 해당 내용이 기록됩니다.
  - IN_BOX_STAGING: 뒤의 문서에서 다루겠지만, 카프카 리스너까지 안전하게 도착해서, In_box 에 성공적으로 저장된 케이스입니다.
  - ...

  - 더 작성해야 하는데, 취업을 위해 별도로 경진대회를 준비 중이라 잠시 멈춰두고 시간이 날때마다 추가로 정리하겠습니다.



## kafka 를 사용하지 않을 경우
kafka 를 교체하거나 점검하는 케이스도 생각해야 합니다. 이 경우 고객사 요청을 접수하는 `front-desk` API 는 EKS 기반으로 배포되어있어야 합니다. 그리고 kafka 로 publish 하는 대신 `kafka_publisher_dead_letters` 에 해당 요청들을 기록합니다.<br/>
<br/>















