# Follow, 댓글/답글 정책
<br/>

# Follow 정책
한 사람이 1000명 이상을 follow 하지 못하도록 막아두었습니다. 예를 들어 나를 팔로우 하는 팔로워가 100만명이 되는 것은 상관이 없지만, 팔로잉의 경우 1000명 이상을 팔로우하는 것은 금지해두었습니다. 즉 999명 까지만 팔로우가 가능합니다. 999명 이상을 팔로우하는 것은 서비스를 악용하는 사용자이거나 bot일 수도 있는 가능성이 있고, 피드를 조회할 때 1000명 이상 멤버에 대한 피드를 조회해야 하는 최악의 케이스가 생기기 때문에 follow 최대 허용 인원수를 999명까지로 제한해두었습니다.<br/>
<br/>

# 댓글/답글 허용 레벨
답글은 3레벨 까지만 작성히 가능하도록 했습니다. 답글 삭제시 중첩된 삭제가 발생할 가능성이 높기 때문에 가급적 3레벨까지 허용하는 것이 맞다고 판단했습니다. 댓글/답글 허용 레벨에 대한 규칙을 마련하면서 reddit, twitter, youtube 의 운영 스타일을 확인해봤습니다. reddit,twitter 와 같은 서비스들의 경우에는 정보 공유와 정보의 소스가 중요하면서 개인의 의견을 중요하는 플랫폼의 경우 댓글/답글 레벨을 무한에 가깝게 되어 있었습니다. youtube 의 경우에는 커뮤니티 비슷한 성격을 가지며 개인의 사생활을 보호하는 플랫폼의 성격을 띄어서 3레벨의 답글까지만 허용되는 것을 확인했습니다.<br/>

dailyfeed 의 경우 커뮤니티의 성격이 더 강하고, 지식의 공유 성격보다는 개인 프라이버시가 더 중요하기에 3레벨 내에서만 소통이 가능하도록 구성했습니다.<br/>
<b/r>

# 댓글/답글 삭제 시 정책
**댓글 삭제 시 자식 답글들은 모두 삭제됩니다. 답글 삭제 시 자식 답글들 역시 모두 삭제됩니다.**<br/>
reddit, twitter 와 같은 정보공유/정보의 소스, 개인의 의견을 중요하게 생각하는 플랫폼의 경우 ‘삭제된 글입니다’로 표시 후 자식 댓글/답글을 표현하지만, dailyfeed 의 경우에는 커뮤니티의 성격이 더 강하고 사생활 측면이 더 중요하다고 느껴 댓글/답글 삭제시 자식 댓글/답글 삭제를 하도록 했습니다.<br/>
<br/>

# 댓글/답글 API 는 별도로 분리
댓글 작성 API 와 답글 작성 API 를 POST /api/comments 에 두는 것도 생각할수 있지만, 경험상 답글의 경우 댓글과 정책이 조금씩 달라지는 케이스가 있기에 댓글 작성 API, 답글 작성 API 는 별도로 분리했습니다. API 하나에서 두 가지 이상의 기능으로 분기를 하는 레거시 코드를 접한 경험이 있기에 두 기능을 별도의 API로 분리해서 독립적으로 수정 또는 관리되도록 구성했습니다.<br/>
<br/>

`dailyfeed-content`/ `CommentController.java`
```java
@Slf4j
@RequiredArgsConstructor
@RequestMapping("/api/comments")
@RestController
public class CommentController {
    private final CommentService commentService;

    ///  /comments  ///
    // 댓글 작성
    @PostMapping({"","/"})
    public DailyfeedServerResponse<CommentDto.Comment> createComment(
            @AuthenticatedMemberProfileSummary MemberProfileDto.Summary member,
            @RequestHeader("Authorization") String authorizationHeader,
            HttpServletResponse httpResponse,
            @Valid @RequestBody CommentDto.CreateCommentRequest request) {
        CommentDto.Comment result = commentService.createComment(member, authorizationHeader, request, httpResponse);
        return DailyfeedServerResponse.<CommentDto.Comment>builder()
                .status(HttpStatus.OK.value())
                .result(ResponseSuccessCode.SUCCESS)
                .data(result)
                .build();
    }
    
    // ... (중략) ...
    
}
```
<br/>

`dailyfeed-content` / `CommentReplyController.java`
```java
@Slf4j  
@RequiredArgsConstructor  
@RequestMapping("/api/comments/replies")  
@RestController  
public class CommentReplyController {  
    private final CommentService commentService;  
  
    /// /comments/replies  
    @PostMapping("")  
    public DailyfeedServerResponse<CommentDto.Comment> createReply(  
            @AuthenticatedMemberProfileSummary MemberProfileDto.Summary member,  
            @RequestHeader("Authorization") String authorizationHeader,  
            HttpServletResponse httpResponse,  
            @Valid @RequestBody CommentDto.CreateCommentRequest request) {  
        CommentDto.Comment result = commentService.createReply(member, authorizationHeader, request, httpResponse);  
        return DailyfeedServerResponse.<CommentDto.Comment>builder()  
                .status(HttpStatus.OK.value())  
                .result(ResponseSuccessCode.SUCCESS)  
                .data(result)  
                .build();  
    }  
}
```