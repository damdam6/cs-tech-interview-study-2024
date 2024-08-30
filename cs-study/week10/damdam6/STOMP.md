# STOMP

Date: August 25, 2024 → August 27, 2024
Status: In progress
Categories: Back-study:cs-study
Tags: Backend

# STOMP

- Stomp
    - Simple/Streaming Text Oriented Messaging Protocol
    - 텍스트 기반의 메시지 프로토콜
    - 규칙 - 클라이언트와 서버 간 전송할 메시지의 유형, 형식, 내용 등을 정의한 규칙으로 TCP 또는 WebSocket과 같은 양방향 네트워크 프로토콜 기반으로 동작
    - 브로커와 클라이언트 간에 메시지를 교환하는 방식으로 동작

- 구조
    - Publish - Subscribe
        - 메시지 공급과 소비 주체를 분리하여 제공함.

- Websocket VS STOMP
    - Websocket
        - 메시징 아키텍처를 의미. TCP 위의 얇은 레이어로 바이트 스트림을 메시지 스트림으로 변환하는 역할을 함.
        - 실제로 구축했던 socket 통신 규칙
            
            ![image.png](/image/image.png)
            
        - HTTP는 애플리케이션 레벨 프로토콜 → but 웹소켓 프로토콜은 프레임워크나 컨테이너가 수신된 메시지를 어떻게 라우팅하고 처리할지 알 수 있을만큼의 정보가 없음.
            
            ⇒ 결국 이 위에 서브 프로토콜을 자체적으로 구축해야 함.
            
        
    - STOMP
        - websocket의 표준화된 서브 프로토콜 중 하나
            - MQTT, AMQP 등 다른 프로토콜도 있음!
    
- STOMP 기반 통신 흐름
    
    ![image.png](/image/image%201.png)
    
    - simpleBroker는 바로 구독자에게 전송
    - 메시징 가공이 필요한 경우 로직을 거쳐서 전달.
    
    - 외부 메시지 브로커를 연결할 경우
        
        ![image.png](/image/image%202.png)
        

- STOMP 프레임
    
    ```jsx
    COMMAND
    header1:value1
    header2:value2
    
    Body^@
    ```
    
    - COMMAND : `SEND` `SUBSCRIBE` `CONNECT` `MESSAGE` 등…
    
    - 프레임 워크의 동작 예시
        - FE
            
            `/app/hello` 경로로 전송
            
            ```jsx
            function sendName() {
                stompClient.send("/app/hello", {}, JSON.stringify({'name': 'John Doe'}));
            }
            ```
            
        - BE
            
            ```jsx
            import org.springframework.messaging.handler.annotation.MessageMapping;
            import org.springframework.messaging.handler.annotation.SendTo;
            import org.springframework.stereotype.Controller;
            
            @Controller
            public class GreetingController {
            
                @MessageMapping("/hello")
                @SendTo("/topic/greetings")
                public Greeting greeting(HelloMessage message) throws Exception {
                    return new Greeting("Hello, " + message.getName() + "!");
                }
            }
            ```
            
        - STOMP 프레임 예시
            
            FE → BE
            
            ```
            SEND
            destination:/app/hello
            content-type:application/json
            content-length:19
            
            {"name":"John Doe"}^@
            ```
            
            - SEND : 클라이언트가 서버로 메시지 보냄
            - destination  :  경로
            - content-type : 메시지의 콘텐츠 타입
            - content-length : 본문 길이
            
            BE → FE
            
            ```
            MESSAGE
            destination:/topic/greetings
            content-type:application/json
            subscription:sub-001
            message-id:msg-123
            content-length:26
            
            {"content":"Hello, John Doe!"}^@
            ```
            
            - subscription : 구독 ID
            - message-id : 식별자
    
- STOMP의 한계점
    - 텍스트 기반 프로토콜 → 바이너리 데이터 전송 등 높은 성능이 요구될 때는 사용 지양
    - 간단한 형태의 구조 → 복잡한 요구사항 처리는 어려움.
    - 메시징의 표준화가 없음 : 채팅 등에는 유리하나 고정적인 메시징이 요구된다면 부적합할 수 있음.
    - 보안기능 제공 X

---

https://stackoverflow.com/questions/40988030/what-is-the-difference-between-websocket-and-stomp-protocols

https://stomp.github.io/stomp-specification-1.2.html#STOMP_Frames

https://jmesnil.net/stomp-websocket/doc/

https://velog.io/@junsu1222/STOMP에-RabbitMQ를-추가해보았다

https://innu3368.tistory.com/213

https://www.youtube.com/watch?v=rvss-_t6gzg

https://docs.spring.io/spring-framework/reference/web/websocket.html