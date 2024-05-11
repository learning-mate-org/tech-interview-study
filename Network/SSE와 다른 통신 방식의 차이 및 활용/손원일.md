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

    private Long TIME_OUT = Long.MAX_VALUE;

    @RequestMapping("/sse")
    public SseEmitter streamSseMvc(@RequestParam Long userId){
        SseEmitter sseEmitter = new SseEmitter(TIME_OUT);
        // 연결 확인
        try{
            sseEmitter.send(SseEmitter.event().name("connect"));
        }catch(IOException e){
            e.printStackTrace();
        }
        sseService.putEmitter(userId, sseEmitter);
        sseEmitter.onCompletion(()->sseService.remove(userId));
        sseEmitter.onTimeout(()->sseService.remove(userId));
        sseEmitter.onError(()->sseService.remove(userId));
    }
// 실시간 데이터를 생성하는 곳
    @RequestMapping("/addComment")
    public ModelAndView addComment(@RequestParam Long sendUserId, Long ReciveUserId, String comment){
        String data = sendUserId +";"+comment;
        sseService.sendData(ReciveUserId, data);
    }
```
```Java
// SSE Service
    @Autowired
    private SseRepository sseRepo;

    public void putEmitter(Long userId, SseEmitter sseEmitter){
        sseRepo.putEmitter(userId, sseEmitter);
    }

    public SseEmitter getEmitter(Long userId){
        try{
            return sseRepo.getEmitter(userId);
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    public void remove(Long userId){
        sseRepo.remove(userId);
    }

    public void sendData(Long userId, String data){
        try{
            SseEmitter sseEmitter = sseRepo.getEmitter(userId);
            sseEmitter.send(SseEmitter.event().name("addComment").data(data));
        }catch(IOException e){
            sseRepo.remove(userId);
        }catch(Exception e){
            e.printStackTrace();
        }
        
    }
```
```Java
// SSE Repository
    private Map<Long,SseEmitter> sseMap = new HashMap<>();

    public void putEmitter(Long userId, SseEmitter sseEmitter){
        sseRepo.put(userId,sseEmitter);
    }

    public SseEmitter getEmitter(Long userId){
        if(sseMap.containsKey(userId)){
            return sseMap.get(userId);
        }else{
            // 예외 발생
        }
        return null;
    }

    public void remove(Long userId){
        sseMap.remove(userId);
    }
```
### 장·단점
- 장점<br/>
&nbsp; 1. HTTP를 통해 통신하므로 다른 프로토콜은 필요가 없고, 구이 쉽다.<br/>
&nbsp; 2. 네트워크 연결이 끊겼을 때 자동으로 재연결을 시도한다.<br/>
&nbsp; 3. 실시간으로 서버에서 클라이언트로 데이터를 전송할 수 있다.<br/>
- 단점<br/>
&nbsp; 1. GET 메소드만 지원하고, 파라미터를 보내는데 한계가 있다.<br/>
&nbsp; 2. 단방향 통신이며, 한번 보내면 취소가 불가능하다는 단점이 있따.<br/>
&nbsp; 3. 클라이언트가 페이지를 닫아도 서버에서 감지하기가 어렵다는 것도 단점이다.<br/>
&nbsp; 4. 지속적인 연결을 유지해야하므로, 많은 클라이언트가 동시 연결을 유지할 경우 서버 부담이 커진다.
### 다른 통신 방식과의 차이점
![image](https://github.com/learning-mate-org/tech-interview-study/assets/67799705/63770819-a73d-4580-9de5-d3389ad2f479)
- 폴링<br/>
&nbsp; 클라이언트에서 서버에게 데이터를 반복적으로 요청하는 방식으로, 서버는 클라이언트에서 요청을 했기 때문에 '의미없는 데이터'도 응답하게 됩니다.<br>
&nbsp;단순한 모델로 구현이 가능하고 일정하게 갱신되는 서버 데이터의 경우 유용하게 활용 가능하다<br/> 
&nbsp; 그러나 클라이언트가 지속적인 데이터 요청을 하기 때문에 HTTP 오버헤드가 발생하고 서버 부담이 크고 실시간 정도의 빠른 응답은 기대하기가 어렵습니다.
- 긴폴링<br/>
&nbsp;폴링 방식의 약간 변형한것으로,클라이언트가 서버에 데이터를 요청하고, 서버는 '의미없는 데이터'를 보내지 않기 위해 새로운 데이터로 갱신 되었을 때 응답하는 방식입니다.<br/>
&nbsp;요청 횟수를 줄여서 폴링의 단점인 HTTP 오버헤드와 서버 부담을 줄일 수 있습니다.<br/>
&nbsp;데이터 갱신 속도가 짧으면 폴링과 별 차이가 없으며 클라이언트가 많아지면 서버에 부담이 발생됩니다. 
- Web Socket 통신<br/>
&nbsp; 웹 소켓 포트에 연결되어 있는 모든 클라이언트에게 이벤트 방식으로 응답하는 방식으로 최초 접속에서 http request를 통한 handshaking 과정을 통해 이루어지고 이후엔 서버에서 데이터 발생시 클라이언트에게 데이터를 보내게 됩니다<br/>
&nbsp; 80, 443포트 접속으로 방화벽이 없고 CORS 적용, 인증 등 동일하게 처리가 가능합니다.<br/>
&nbsp; websoket 프로토콜을 위한 전이중 연결과 웹소켓 서버가 필요합니다.
### 이점 및 활용
&nbsp; 단방향 통신과 실시간의 이점을 살린 통신이기 때문에 SNS에서의 구독 알림이나, 좋아요 알림에서 활용하거나, 실시간으로 데이터를 얻어와 표나 그래프를 갱신 시킬때 활용할 때 SSE 장점을 잘 활용할 수 있다고 생각합니다.