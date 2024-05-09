# SSE
### SSE란?
- 서버에서 클라이언트로 실시간 이벤트를 전달하는 웹 기술로 '단방향'통신입니다.
- 실행과정<br/>
&nbsp; 1. 클라이언트가 서버의 이벤트를 구독하기 위한 요청을 보낸다<br/>
&nbsp; 2. 서버에서는 클라이언트와 매핑되는 SSE객체를 만든다
&nbsp; 3. 서버는 이벤트 스트림을 생성하고 클라이언트에게 비동기적으로 데이터를 전송한다.
```JavaScript
// 클라이언트에서 서버에게 구독 요청을 보내는 구문
// EventSource 생성자에게 접근할 url을 입력해 주어야합니다
const eventSource = new EventSource('/sse='+userId);
eventSource.addEventListener("addComment", (event)=>{
    // event.data를 통해 서버에게서 전달받은 이벤트 정보를 받아 올수 있습니다
    let [userId, comment] = event.data.split(";");
    $("#commentBox").append("<div>"+
                                "<div>"+userId+"</div>"+
                                "<div>"+comment+"</div>");
}); 
```
```Java
/** Spring boot를 활용하였습니다
 * 서버에서 클라이언트와 매핑되는 SSE객체를 생성하고
 * 이벤트 스트림을 생성하여 비동기적으로 데이터를 전송합니다. 
*/  
// SSE Controller
    @Autowired
    private SseService sseService;

    @RequestMapping("/sse")
    public SseEmitter streamSseMvc(@RequestParam Long userId){
        SsseEmitter sseEmitter = 
    }
```
```Java
// SSE Service
    @Autowired
    private SseRepository sseRepo;
```
```Java
// SSE Repository
    private Map<Long,SseEmitter> sseMap = new HashMap<>();


```
### 통신 방식과의 차이점
- 폴링
- 긴폴링
- Socket 통신
### 이점 및 활용
