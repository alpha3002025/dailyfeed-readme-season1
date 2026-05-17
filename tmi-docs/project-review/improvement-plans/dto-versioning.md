Dailyfeed Code 를 구성하게 된 계기는 "Dto 객체는 많아질 수록 관리가 힘들다" 라는 생각에서 출발했고, "어떻게 하면 효율적으로 관리할 수 있을까?" 하는 의구심에서 출발했었습니다. 그리고 차기버전에서는 Dto 하나에 여러 기능에 대한 메시지의 인터페이스를 모아 두는 것으로 인한 의존성과 결합도가 높아진 상황을 어떻게 개선할지에 대해 세운 계획을 이번 문서를 통해 정리하고자 합니다. 이번 문서에서는 Dailyfeed 내의 Dto 중 Post 도메인을 예를 들어서 설명합니다. 다른 도메인에 대해서도 해당 내용을 적용할 예정입니다.<br/>
<br/>

## PostDto 의 형태
dailyfeed-code 에서 dto 는 일반적으로 PostDto.java 와 같은 클래스에 다음과 같은 내부 클래스들을 static 하게 두었습니다.

dailyfeed-code/PostDto.java
```java
// ...

public class PostDto {
    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class CreatePostRequest {
	    // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class UpdatePostRequest {
	    // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostsBulkRequest {
        private Set<Long> ids;
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class Post{
        // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostSummary {
        // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostActivityEvent { // MongoDB에서 읽어들일때는 PostActivityType = DELETED 가 아닌 데이터 + followingId = ?, Sort = UpdatedAt DESC 을 Paging
        // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class LikeActivityEvent { // MongoDB에서 읽어들일때는 PostActivityType = DELETED 가 아닌 데이터 + followingId = ?, Sort = UpdatedAt DESC 을 Paging
        // ...
    }

    /// 통계
    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostLikeCountQueryBulkRequest {
        private Set<Long> postPks;
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostLikeCountStatistics {
        // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostCommentCountQueryBulkRequest {
        private Set<Long> postPks;
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostCommentCountStatistics {
        // ... 
    }

    /// 삭제 예정 //////
    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostSearchResult {
        // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostSearchCriteria {
        // ...
    }

    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class PostStatistics {
        // ...
    }
}
```

이 PostDto 내에는 데이터 응답을 위한 일반적인 형태의 응답객체인 Post, PostSummary 등과 같은 일반 응답객체, CreatePostRequest, UpdatePostRequest, PostsBulkRequest, PostCommentCountQueryBulkRequest 등과 같은 Request 객체, PostStatistics, PostCommentCountStatistics 와 같은 통계 관련 응답을 답는 데이터 객체, PostSearchCriteria 와 같은 검색 조건 객체, PostActivityEvent, LikeActivityEvent 와 같은 kafka 메시징 객체 등이 있습니다.

PostDto
- 데이터 응답을 위한 일반적인 형태의 응답객체인 Post, PostSummary 등과 같은 일반 응답객체
- CreatePostRequest, UpdatePostRequest, PostsBulkRequest, PostCommentCountQueryBulkRequest 등과 같은 Request 객체
- PostStatistics, PostCommentCountStatistics 와 같은 통계 관련 응답을 답는 데이터 객체
- PostSearchCriteria 와 같은 검색 조건 객체
- PostActivityEvent, LikeActivityEvent 와 같은 kafka 메시징 객체


## dailyfeed-code 로 분리한 이유

kafka 등을 통해 메시징을 하거나 feign 을 이용해서 다른 서비스간에 통신을 하려면 서로 간에 공통된 형식, 인터페이스가 필요합니다. 이때 가장 간단한 방법은 각각의 서비스에 객체를 구성해서 형식을 맞춰주면 됩니다. 그런데 변경사항이 자주 발생하거나 서비스 간에 서비스 운영 일정이 조금씩 다르기 때문에 객체의 구성이 달라질 수 있습니다. 따라서 각각의 서비스가 동일한 인터페이스를 바라봐야 하는데, 이때 dailyfeed-code 라는 gradle 모듈에 해당 내용을 정의해두고 이 것을 각각의 서비스가 공유할 경우 객체 수정시에 발생하는 데이터의 불일치 문제를 해결할 수 있습니다. 이런 이유로 dailyfeed-code 모듈을 도입하게 되었습니다.

그런데 이 방식에도 문제가 있긴 합니다. 카프카를 사용할 경우 버전이 올라갈 때마다 발신/수신측의 애플리케이션 버전이 달라질때 생기는 결합도와 불일치 문제입니다.<br/>
<br/>

### (보완해야 한다고 느낀 점) 발신/수신측 데이터 불일치 문제
이 경우 한쪽이 개발이 완료된 후까지 dailyfeed-code 의 수정을 미뤄두고 기다려야 한다던가 특정 필드는 Transient 하게 하거나, 버전을 다르게 해줘야 하거나, 배포가 어려워질 수 있는 케이스가 생길 수 있습니다. 즉, 서로 바라보는 객체내의 필드의 구성이 달라지거나, 어떤 필드는 존재하지 않게 되는 경우가 있을 수 있고, 이미 운영환경에 있는 앱이 어떻게 동작하게 될지를 고려해 복잡한 대응을 해야 하는 케이스가 생길수 있습니다. 그리고 통신의 애매모호함이 기능 수정 후 배포 시에 어려움과 스트레스로 다가오게 됩니다.<br/>
<br/>

이런 단점은 패키지를 용도별로 분류하고 해당 패키지 내의 서브 패키지 명을 v1, v2 등으로 분류해서 버전별 통신 인터페이스를 다르게 가져가는 방식을 고려해볼 수 있습니다.<br/>
<br/>

PostDto 를 예로 들면, 향후 개발할 차기버전의 Feed 앱의 경우 Dto 를 다음과 같이 구분해서 Dto 객체들을 분류해나갈 예정입니다.<br/>
- internal
	- dailyfeed-code/domain/content/post/dto/internal/v1,v2,.../
		- e.g.
		- CreatePostRequest, UpdatePostRequest, PostsBulkRequest, PostCommentCountQueryBulkRequest
- kafka
	- dailyfeed-code/domain/content/post/dto/kafka/v1,v2,.../
		- PostMessageDto 내에 다음의 static inner 클래스들을 추가
			- e.g.
			- PostCreateEvent, PostUpdateEvent, PostDeleteEvent
			- PostActivityEvent, LikeActivityEvent
- feign
	- dailyfeed-code/domain/content/post/dto/feign/v1,v2,.../
		- PostFeignDto 내에 다음의 static inner 클래스들을 추가
			- e.g.
			- CreatePostRequest, UpdatePostRequest, PostsBulkRequest, PostCommentCountQueryBulkRequest
- ... 


이렇게 도메인에 대한 객체를 용도별로 1차 분류 후 버전별로 2차 분류를 해서 다른 서비스들이 하나의 통신객체를 바라보는 것으로 인한 결합도를 낮출수 있습니다. 초기 버전은 예제의 간결성과 직관적인 예제를 위해 하나의 객체에 모아두었지만, 차기 버전부터는 위와 같이 용도별, 버전별로 Dto 객체들을 분류해서 관리를 해나갈 예정입니다.<br/>

이 방식의 버저닝 계획은 추후 다른 도메인에 대한 객체들에도 동일하게 적용할 예정입니다.<br/>
<br/>

