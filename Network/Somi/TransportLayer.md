# 🔃 응용계층 Review

- TLS : HTTPS를 이용하는 웹 사이트가 사용하는 보안 프로토콜
    - 사용자가 브라우저 사용창에 https://www.naver.com 입력하는 순간 데이터 날아가기 전 ( TCP 3-way Handshake → TLS Handshake ) 가 선행됨
    - **TLS Handshake**
        1. ClientHello : 클라이언트가 먼저 서버에게 메시지 전송
            1. 가능한 TLS 버전, 암호화 방식, 난수 알려줌
        2. ServerHello : Client Hello에 대한 응답
            1. 가장 안전한 방식을 택함
        3. ServerCertificate : 서버의 인증서를 클라이언트에게 보냄
        4. ServerHelloDone : 서버가 클라이언트에게 보낼 메시지 끝남
            1. 내 소개는 끝났어
        5. ClientKeyExchange
            1. 인증서 무결한지 검증, 클라이언트-서버의 난수를 조합하여 대칭키 생성
            2. 서버 인증서 속 공개키로 대칭키를 암호화해서 보냄 ( 앞으로 쓸 거임 )
            3. 서버로 보냄 ( 서버의 개인키로만 이 대칭키를 풀 수 있음 )
        6. ChangeCipherSpec : 이제부터 패킷은 협상된 알고리즘,키를 이용해 암호화하겠다고 알림
        7. Finished
- 대칭키 : 암호화, 복호화에 같은 키 사용하는 알고리즘
- 비대칭키/공개키 : 암호화, 복호화할 때의 키가 서로 다른 알고리즘
    1. B가 공개키,개인키 쌍 만들어서 공개키를 알림
    2. A는 B의 공개키로 암호화 후 전송
    3. B는 자신의 개인키로 복호화
- CSRF ( Cross Site Request Forgery )
    
    : 인증된 사용자가 자신의 의지와는 상관없이 웹사이트에 공격자가 의도한 요청을 보내도록 유도하는 것
    
    EX) 은행 웹사이트에서 인증된 상태로 로그아웃을 하지않고 악의적인 사이트 방문, 사용자가 어떤 html 클릭하는 순간 자금이체가 진행됨.
    
- SOP ( Same-Origin Policy : 동일출처정책 )
    
    : 웹 브라우저 보안을 위해서 same origin ( 프로토콜, 호스트, 포트가 같은 ) 서버로만 리소스를 주고 받도록 제한하는 보안 방식
    
- Http Status Code
    
     1XX : 요청이 수신되어 처리중 
    
    2XX : 요청 정상 처리 
    
    3XX : 요청을 완료하려면 추가 행동이 필요 
    
    4XX : 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음 
    
    5XX : 서버 오류, 서버가 정상 요청을 처리하지 못함
    
- 웹소켓과 http의 차이
    
    HTTP는 요청을 보내면 응답이 오는 단방향적 구조로 통신
    
    웹소켓은 양방향 통신이 가능함
    
    비연결성이라는 특징을 가진 HTTP와 다르게 웹소켓은 한번 연결 맺은 뒤 유지됨
    

---

# 1. 전송 계층

- 역할
    
    네트워크 양 끝단에서 통신을 수행하는 호스트 간의 단대단( End-to-End ) 연결 제공
    
    네트워크 계층에서 제공하는 서비스를 이용해 상위계층의 사용자에게 신뢰성있는 서비스 제공
    
    : 출발지 - 도착지까지 패킷이 제대로 전송될 수 있도록함
    
    : 응용계층에서 만든 데이터를 일정한 크기로 자름 ( 패킷 단위로 )
    

→ 응용 계층의 앱들이 전송계층의 프로토콜을 이용하기 위해서는 프로세스마다 소켓을 이용해야함. 

→ 컴퓨터에는 여러 앱들이 돌아가고, 앱은 하나 이상의 소켓을 생성할 수 있음

→ 전송 계층은 여러 소켓에서 쏟아지는 데이터들이 제대로 전송될 수 있도록 해야함

## Multiplexing ( 소켓의 데이터를 패킷으로 )

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2F5pC0M%2FbtqVFBFjjhG%2FAAAAAAAAAAAAAAAAAAAAAGWKRSvzc2w7XRnS9GTBk5cGJLwFJ7uVgCsuFL6B-LFy%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3DyBmHK5jpIQdeSGPUuu1XHMgQ6%252Bs%253D">
: 여러 앱들의 소켓들로부터 들어오는 데이터를 수집 → 패킷으로 만들어 → 하위레이어로

목적지가 동일할 경우 데이터들을 하나의 가상회선에 실어서보냄

## Demultiplexing ( 받은 패킷을 소켓으로 )

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fb4daRi%2FbtqVIb7pDsM%2FAAAAAAAAAAAAAAAAAAAAACmIFLw1ao3XZC349sj4lkagAyFuSraEcqeN4iltbs8V%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3DVBfLz%252F3JEQzuarXmYl9ZIUXGtew%253D">
: 하위레이어로부터 들어온 패킷을 → 올바른 소켓으로 전송(→ 올바른 앱으로 전송 )

( 이때, 정확한 앱의 소켓으로 전달해주려고 포트번호를 이용하는 것 ! )

## vs 데이터링크 계층

<img src="https://velog.velcdn.com/images/jjh0526/post/59f7eac2-e2f8-4527-bf8e-accfbd59b9bd/image.png">
- 데이터링크 계층 : **인접 노드** 간 신뢰성을 보장함
    - 물리적으로 연결되어 있는 노드 간 오류제어, 흐름제어, 회선제어 제공함
- 전송 계층 : **종단** 간 신뢰성을 보장함
    - 흐름제어, 혼잡제어, 오류제어 제공함

> ✅
> 
> 
> 전송계층은 **논리적** 연결 ( 종단간 호스트 )
> 
> 데이터링크 계층은 **물리적** 연결 ( 인접한 노드 or 호스트 )
> 

# 2. 포트, 소켓, 웹소켓 ( + HTTP )

## 포트

논리적 접속장소

: TCP/IP 사용할때 클라이언트 프로그램이 네트워크 상의 특정 서버를 지정할 때 사용함
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fbla9yS%2FbtrNvyQx2a0%2FAAAAAAAAAAAAAAAAAAAAANJ6jf8k8XlLG6a7vmQ-DT4h7sUY3IZFs0nkx179XGdP%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3DFoxLV8SCQaCks0e%252FPSe%252FqEM7J3A%253D">

1. IP주소를 바탕으로 서버가 포함되어 있는 컴퓨터에 도착 

( 하나의 IP주소에 여러 애플리케이션을 실행할 경우, 한 컴퓨터의 여러개의 서버가 실행 )

1. 포트번호를 통해 특정 애플리케이션의 서버에 접속할 수 있는 것

| 번호 | 프로토콜 | 통신 프로토콜 | 설명 |
| --- | --- | --- | --- |
| 80 | HTTP | TCP | 웹 서버 접속 |
| 443 | HTTPS | TCP | 웹 서버 접속 ( SSL ) |

→ 웹소켓이 

사용하는 포트 번호

## 소켓

네트워크를 경유하는 프로세스 간 통신의 종착점

: 응용 계층에 속하는 프로세스들은 데이터 송수신을 위해 반드시 소켓을 거쳐 전송 계층으로 데이터 보냄

: 전송 계층과 응용 프로그램 사이의 인터페이스 역할을 하며 떨어져 있는 호스트를 연결

- 아래의 요소를 담은 하나의 파일
    - 프로토콜
    - 상대와 자신의 IP 주소
    - 포트 번호
    - 통신 동작 진행 상태
- 종류
    - 스트림 소켓 : TCP 방식을 사용하는 연결지향 방식의 소켓
        - 데이터 순서 보장 / 점대점 연결 / 대량 데이터 전송에 적합
    - 데이터그램 소켓 : UDP 방식을 사용하는 비연결 방식의 소켓
        - 데이터 순서와 신뢰 보장X / 일대다 연결도 가능 / accpet()없이 바로 전송

## 웹소켓 ( vs HTTP )

웹 브라우저 환경에서 서버와 **지속적인 양방향 통신**을 하기 위해 만들어진 프로토콜

- **vs HTTP ?**

기존 HTTP는 요청-응답 구조만 가지고 있었음

따라서, 웹 브라우저가 서버로부터 실시간 데이터를 받고 싶다면 지속적으로 새 요청을 보내거나, 롱 폴링 ( long polling ) 이라는 방식을 사용해야함

---

💡 long polling : polling이 일정 주기마다 요청을 보내는 방식이라면, long polling 은 비슷하지만 서버가 요청마다 바로 응답을 전달하지 않고 특정 이벤트, 타임아웃이 발생했을 때 응답 전달하는 방식

---

**→ ⁉️ 이 방식을 사용하니 네트워크 오버헤드가 발생하고 실시간 통신에 적합 X**

하지만 웹 브라우저는 보안을 위해 TCP로 직접 소켓을 연결할 수 없었음.

✅ 따라서 브라우저에서 사용할 수 있는 안전한 TCP 소켓이 필요 —> **웹소켓** !!

<img src="https://i.imgur.com/t56epcK.png">
→ 웹소켓은 HTTP를 통해 연결을 수립

→ 연결된 후부터 TCP 소켓으로 실시간 양방향 통신 지원

> ⁉️ 소켓과 웹소켓의 차이점
> 
> 
> ✅ **웹 소켓은 TCP 소켓과 구분되는 것이 아니라 TCP 소켓의 추상화된 형태이다.**
> 
> 소켓 통신에 기반하여 웹 소켓은 웹 애플리케이션에 맞게 발전한 형태로 소켓 통신을 한다!
> 

# 3. TCP, UDP

💡 프로토콜이란 , 클라이언트-서버가 정보를 교환할 수 있도록 하는 메시지 형식 규칙

## TCP ( 전송 제어 프로토콜 )

Transmission Control Protocol

: 서버와 클라이언트간에 데이터를 신뢰성 있게 전달하기 위해 만들어진 프로토콜

- **TCP의 신뢰성 보장**
    
    : 패킷 내부의 인증 Flag 값들을 확인해서 클라이언트-서버가 서로 보낸 패킷, 패킷의 순서가 올바른지 검증하는 것
    
    > **⁉️ TCP의 신뢰성 보장 방식**
    > 
    > 
    > ✅ 3-way로 통신 시작할때, 4-way로 통신을 마칠 때 모두 handshake를 진행하며 신뢰성을 극대화함. 통신을 시작하거나 종료할때 상대방의 준비 상태를 반드시 확인하고, 패킷의 전송 순서를 확정한 후에야 데이터를 교환함
    > 
    - Flag의 종류
    
    | FLAG |  |
    | --- | --- |
    | SYN | 접속요청 시 보내는 패킷을 뜻함 |
    | ACK | 상대방으로부터 패킷 받은 후 잘 받았다고 알려주는 패킷을 뜻함 |
    | PSH | 데이터를 즉시 목적지로 보내라는 의미 |
    | FIN | 접속종료를 위한 플래그 |
    
    ---
    
    💡 패킷 ( Packet ) : 패키지 ( package ) + 덩어리 ( bucket )
    
    → 통신망을 통해 전송하기 쉽도록 데이터를 잘게 나눈 전송 단위
    
    ---
    
    ### 1 ) 3 Way Handshake ( 통신 시작 )
    
    <img src="https://velog.velcdn.com/images/yeonjin1357/post/16174726-283c-40b4-b10c-6acad71f1d16/image.png">
        
    1. 클라이언트 : 접속 요청하는 SYN 패킷 보냄 
    2. 서버 : SYN 요청받으면 ACK 패킷과 SYN 패킷을 보냄
        
        ( 서버도 클라이언트에게 접속을 요청해야하기 때문에 SYN을 같이 보냄 )
        
    3. 클라이언트 : 서버에게 다시 ACK 패킷을 보냄
    
    ### 2 ) 데이터 통신 과정
    
    <img src="https://velog.velcdn.com/images/yeonjin1357/post/eb4ed245-7845-4adf-91aa-ade97d97e4d1/image.png">
        
    1. 클라이언트 : 서버에게 데이터 보냄
    2. 서버 : 잘 받았다고 ACK 패킷으로 응답
        1. 클라이언트가 서버로부터 ACK을 받지 못하면 제대로 전송되지 않았다고 판단, 재전송
    
    ### 3 ) 4 Way Handshake ( 통신 종료 )
    
    <img src="https://velog.velcdn.com/images/yeonjin1357/post/376e9588-989c-45cb-b99b-9eea0790d9cd/image.png">
        
    1. 클라이언트 : TCP 연결을 끊기 위해 CLOSE() 함수를 호출 ,FIN 패킷 보냄
    2. 서버 : ACK 패킷을 보냄
        1. 서버에서 클라이언트로 보낼 데이터가 남은 경우, 이때 나머지를 전송함
    3. 서버 : 서버가 CLOSE() 함수를 호출, FIN 패킷을 클라이언트에게 보냄
    4. 클라이언트 : 서버로부터 연결 닫았다는 FIN 패킷 받으면 ACK을 보냄
    5. CLOSED
    
    ---
    
    **💡 TIME WAIT 상태**
    
    : 소켓이 바로 사라지지 않고 일정 시간 유지함
    
    지연 패킷, 두 장치 간 접속 오류 등등 의도치 않은 에러로 인해 연결이 데드락으로 빠지는 문제 방지
    
    ( 데드락 : 2개 이상의 프로세스가 자원을 획득하려고 서로 무기한 대기하는 상황 )
    
    ---
    
    > **⁉️ 왜 연결은 3-way고 해제는 4-way ??**
    > 
    > 
    > ✅ 클라이언트가 데이터 전송을 마쳤다고 해도 서버는 아직 보낼 데이터가 남아있을 수 있기 때문에, ACK만 보내고 → 확인 ( 데이터 모두 전송 ) → FIN 메시지
    > 
    
- **TCP의 전송 제어 기법**
    
    ⁉️ TCP는 네트워크 상태에 따라 데이터 전송량을 제어하는 알고리즘이 있음
    
    ### ✅ 흐름제어 ( Flow Control )
    
    송신-수신 측의 데이터 처리 속도 차이를 해결하기 위한 기법
    
    > receiver가 패킷을 받을 수 있는 버퍼의 크기가 정해져있는데,
    > 
    > 
    > sender 측의 전송 속도가 너무 빨라 수많은 패킷을 한번에 받아버리면, 
    > 
    > 버퍼가 가득 차서 손실되는 패킷이 발생함 
    > 
    > —> 따라서 어느 정도의 데이터를 전송할지 제어
    > 
    - **슬라이딩 윈도우 ( Sliding Window ) 방식**
        
        <img src="https://velog.velcdn.com/images%2Fhaero_kim%2Fpost%2F193e1e14-c3bb-4049-a07f-df0a2c9de843%2F253F7E485715ED5F27.png">
                
        : window라는 데이터를 담는 공간을 동적으로 조절해 데이터량 조절함
        
        1. receiver의 window 크기에 맞게 sender가 자신의 window 크기를 조절
            
            ( 데이터 보내기 전 3-way handshake 할 때 조절 )
            
        2. window 크기만큼 패킷 전송 후
        3.  전달이 확인되는대로 윈도우를 옆으로 슬라이딩 하며 다음 패킷 전송함
    - **Stop-And-Wait 방식**
        
        : 매번 전송한 패킷에 확인 응답을 받아야만 그 다음 패킷을 전송
        
    
    ### ✅ 혼잡제어 ( Congestion Control )
    
    ↔ 흐름제어는 송신-수신 사이의 패킷 수 조절
    
    **네트워크 내**의 패킷 수를 조절하여 네트워크의 overflow를 방지하는 방법
    
    > EX) 한 라우터에 데이터가 몰려서 모든 데이터를 처리할 수 없는 상황
    > 
    > 
    > 보낸 데이터에 응답이 오지 않으면, 동일 데이터를 재전송
    > 
    > 재전송 때문에 혼잡이 가중되어 오버플로우, 데이터 손실이 발생
    > 
    > —> 따라서 sender의 데이터 전송 속도를 제어
    > 
    - **AIMD ( Addictive Increase Multicative Decrease ) 방식**
        
        <img src="https://velog.velcdn.com/images/yeonjin1357/post/35397741-0943-4fa8-97f0-2fe3d6c87f1c/image.png">
                
        : 합 증가 - 곱 감소 알고리즘
        
        1. sender가 하나의 패킷을 전송함
        2. 패킷이 문제없이 도착하면, window 사이즈를 하나 증가시켜 두개 보냄
        3. ~ 반복하다가, 패킷 전송되지 않거나, TIME_OUT 발생시 window 사이즈를 절반으로 줄임
        
        > **⁉️ AIMD 알고리즘으로 모든 호스트가 공평하게 네트워크를 사용한다**
        > 
        > 
        > ✅ 네트워크가 혼잡하여 응답이 잘 이루어지지 않는다고 가정했을때, 
        > 
        > 이미 존재하던 호스트는 큰 윈도우 사이즈를 가질 것임.
        > 
        > 새로 들어온 호스트는 작은 윈도우 사이즈를 가질 것임.
        > 
        > 존재하던 호스트는 자신의 큰 윈도우 사이즈로 인해 무리하게 많은 데이터를 재전송 요청하게 되어 데이터를 잃고, 점점 윈도우 사이즈를 줄여감
        > 
        > 새로 들어온 호스트는 자신의 윈도우 사이즈를 하나씩 늘려 남는 대역폭을 사용하게 됨.
        > 
        > 결국은 여러 호스트들이 대역폭을 공평하게 나눠쓰는 것 !
        > 
    - **Slow start 방식**
        
        ↔ AIMD는 천천히 윈도우를 늘려가기 때문에 혼잡상황이 발생하기 전에도 속도를 내지 못하는 단점이 있음
        
        : 시작부터 빠르게 윈도우를 증가시키고 특정 시기에 윈도우를 확 줄여버림
        
        1. sender가 하나의 패킷을 전송함
        2. 패킷이 문제없이 도착하면, window 사이즈를 두배씩 증가시켜 보냄
        3. ~ 반복하다가, 패킷 전송되지 않거나, TIME_OUT 발생시 window 사이즈를 1로 확 줄임

---

**⁉️ TCP는 구조상 한계가 있다**

💡 HOLB ( Head of line Blocking ) 현상

TCP를 사용하면 패킷은 무조건 !!**순서대로**!! 처리되어야함

이전에 받은 패킷을 파싱하기 전까지는 다음 패킷을 처리할 수 없음

손실된 경우는 패킷을 다시 보내야함

→ 이렇게 패킷이 중간에 유실되거나, 받는 쪽의 패킷 파싱 속도가 느리면 병목현상 발생  == HOLB 현상

---

## UDP ( 사용자 데이터그램 프로토콜 )

: 비연결형 프로토콜, 연결 설정 없이 데이터를 전송함

→ 실시간 영상 스트리밍과 같은 고용량 데이터 처리에 이용

## UDP VS TCP

- **TCP는 신뢰성이 높고 느리다, UDP는 신뢰성이 낮고 빠르다**
- 패킷 교환 방식
    - TCP : **가상회선**  ( = 논리적 연결 )  **패킷 교환 방식**
                
        <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FohZuj%2FbtrWQehRqKD%2FAAAAAAAAAAAAAAAAAAAAAKbbCbYpCT72hhKKbZZ57XZBU-NwVrBUbxbdOyP_Btrj%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3DD3QsLQ47y5RnziaEKB4uuM34E6w%253D">
        
        : 패킷 사이 순서 보장
        
        - 각 패킷에는 가상회선 식별자가 포함되어 있음
        - 모든 패킷을 전송하면 가상회선이 해제되고 패킷들이 전송된 **순서대로** 도착하는 방식
    - UDP : **데이터그램 패킷 교환 방식**
        
        <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fd2pbwq%2FbtrWRPVgHnt%2FAAAAAAAAAAAAAAAAAAAAAAIuFqSniPSMRWO1-PKP2F4A-hI3JkUvX4oHEqBfZXGw%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3Di0m6jzAhpSH%252BiHQRiwthvopDt2I%253D">
                
        : 패킷 사이 순서 보장 X, 패킷이 독립적으로 최적의 경로를 선택하여 이동
        
        - 데이터 전송 전에는 논리적 연결이 설정되지 않음
        - 하나의 메시지에서 분할된 여러 패킷은 서로 다른 경로로 전송될 수 있음
        

# ⁉️ 예상 질문 모음

> **✅ 기술 면접 답변틀**
> 
> 1. 기술의 정의 
> 2. 어디에서 사용하는지
> 3. 어떻게 사용할 수 있는지

<aside>
💡

전송계층에 대해서 설명해주세요.

소켓(Socket)이 뭔가요?

소켓통신과 Http통신의 차이점은 무엇인가요?

포트 번호는 왜 필요한가요 ? (어플리케이션 구분이라는 본질을 짚는지 확인) 

- 포트 번호는 몇 번부터 몇 번까지 존재하나요?
- 80번, 443번 포트의 의미는 무엇인가요?
- 동적 포트(dynamic port)와 정적 포트(static port)의 차이를 아시나요?
- 포트 충돌이 발생하면 어떻게 처리하나요?

---

**TCP/UDP**

TCP와 UDP의 차이를 설명해보세요.

- 그럼 TCP는 어떤 방식으로 신뢰성을 보장하나요?
- TCP의 흐름 제어나 혼잡 제어는 각각 어떻게 작동하나요?
- UDP는 왜 실시간 통신에서 자주 쓰이나요?
- UDP를 썼을 때 데이터 유실이 나면 어떻게 처리하나요?

3 way-handshake와 4 way-handshake를 설명해주세요.
TCP 3,4 way handshake의 단계수 차이가 나는 이유
ISN을 0부터 시작하지 않고 난수로 생성하여 설정하는 이유

TIME WAIT의 존재 이유

TCP 통신에서 지연 패킷이 발생했을 때 서버는 이것을 어떻게 처리하나요?

만약 Server에서 FIN 플래그를 전송하기 전에 전송한 패킷이 Routing 지연이나 패킷 유실로 인한 재전송 등으로 인해 FIN 패킷보다 늦게 도착하는 상황이 발생하면 어떻게 될까요?

UDP는 항상 신뢰성을 보장하지 않나요?

→UDP는 신뢰성이 없는게 아니라 탑재를 안했을 뿐이다. UDP의 진짜 장점은 커스터마이징이 가능하다는 점이다.즉, UDP 자체는 헤더에 들은게 없어 신뢰성이 낮고 제어 기능도 없지만, 이후 개발자가 애플리케이션에서 구현을 어떻게 하냐에 따라서 TCP와 비슷한 수준의 기능을 가질 수도 있다는 말이다.

TCP의 3-way handshake에 대해 설명해보세요. (연결 수립 과정을 정확히 말하는게 중요)

- SYN 플러딩 공격이 뭔지 아시나요?
- 연결을 끊을 때는 왜 4단계로 진행되나요?
- Half-open connection이란 무엇인가요?
- 만약 마지막 ACK 패킷이 유실되면 어떤 일이 벌어지나요?

TCP의 흐름 제어와 혼잡 제어는 어떻게 다른가요 ? 

- 흐름 제어는 어떤 기법으로 이루어지나요?
- 슬라이딩 윈도우에 대해 설명해보세요
- 혼잡제어에서 AIMD는 무엇인가요?
- 윈도우 크기는 어떻게 조절 되나요 ?

데이터가 여러개로 나눠져 전송되면 순서를 어떻게 보장하나요 ? (TCP의 순서 보장 메커니즘을 아는지 체크)

- TCP 세그먼트 구조에 대해 설명해보세요
- 중복 ACK가 들어오면 TCP는 어떻게 반응하나요?
- 순서가 뒤바낀 세그먼트가 도착하면 어떻게 처리하나요?
</aside>
