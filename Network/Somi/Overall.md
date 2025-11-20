# 네트워크란 ?

: Net + Work 의 합성어로써 컴퓨터들이 통신 기술을 이용하여 그물망처럼 연결된 통신 이용 형태를 의미

== 두 대 이상의 컴퓨터들을 연결하고 서로 통신(이야기)할 수 있는 것

# OSI 7계층
<img src="https://img1.daumcdn.net/thumb/R1600x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FcgJ1bs%2FbtruwIivAHg%2FAAAAAAAAAAAAAAAAAAAAABmpcro5CDBDMpN_bRaV18SRE9zZbGQJJnAAnfURSemR%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3DVnvwyrcehb1kW7e%252FAoDycGtxVyg%253D">

| OSI 계층 | 역할 | PDU |
| --- | --- | --- |
| L1 ( 물리 계층 ) | 데이터를 전기적인 신호로 변환해서 주고받음 | 비트 |
| L2 ( 데이터 링크 계층 ) | 정보의 오류와 흐름을 관리하여 안전한 정보의 전달을 수행, 통신에서의 오류 찾아주고 재전송 | 프레임 |
| L3 ( 네트워크 계층 ) | 경로(Route)와 주소(IP)를 정하고 패킷을 전달 | 패킷 |
| L4 ( 전송 계층 ) | 신뢰성있는 데이터를 주고 받게 해주는 역할, 오류검출 및 복구, 흐름제어와 중복검사 | 세그먼트 |
| L5 ( 세션 계층 ) | 응용 프로세스가 통신을 관리하기 위한 방법 정의, TCP/IP 세션을 만들고 없애는 역할 | 데이터 |
| L6 ( 표현 계층 ) | 데이터의 표현방식을 결정, 데이터의 암호화와 복호화 | 데이터 |
| L7 ( 응용 계층 ) | 우리가 사용하는 응용 서비스나 프로세스 동작 | 데이터 |

( * PDU 란? : Protocol Data Unit, 각 계층에서의 데이터 단위 )

### OSI 모델의 통신 과정

- **캡슐화** : 응용 계층 -헤더추가→ 물리 계층 ( 송신 )
    - L7 응용 계층 : 사용자 데이터 생성
    - L6 표현 계층 : 데이터 암호화, 압축 수행
    - L5 세션 계층: 세션 정보 추가
    - L4 전송 계층: TCP/UDP 헤더 추가 (세그먼트 생성)
    - L3 네트워크 계층: IP 헤더 추가 (패킷 생성)
    - L2 데이터 링크 계층: MAC 주소 등 프레임 헤더와 트레일러 추가 (프레임 생성)
    - L1 물리 계층: 비트열을 전기 신호로
    
- **역캡슐화** : 물리 계층 -헤더분석→ 응용 계층 ( 수신 )
    - L1 물리 계층: 전기 신호를 비트열로
    - L2 데이터 링크 계층: 프레임 헤더와 트레일러 제거, 오류 검사
    - L3 네트워크 계층: IP 헤더 제거, 라우팅 정보 확인
    - L4 전송 계층: TCP/UDP 헤더 제거, 포트 정보 확인
    - L5 세션 계층: 세션 정보 확인
    - L6 표현 계층 : 데이터 복호화, 압축 해제
    - L7 응용 계층 : 사용자 데이터 수신 및 처리

# TCP/IP 4계층

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FWK1Ct%2FbtsMa5VYPtv%2FAAAAAAAAAAAAAAAAAAAAAL2j9tdJ-nGjwUXqP110cAaPt1cWZGCoIKvsatZuqAR-%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3DjgkB2D%252FiHAyXfo5Rs6f1UhjAvDI%253D">

: 실제로 OSI 7계층보다 활용이 많음, OSI 7계층을 4계층으로 단순화

| TCP/IP 4계층 | 역할 |
| --- | --- |
| L1 ( 네트워크 액세스 계층 ) | L1 ( 물리 계층 ) + L2 ( 데이터 링크 계층 ) |
| L2 ( 인터넷 계층 ) | L3 ( 네트워크 계층 ) |
| L3 ( 전송 계층 ) | L4 ( 전송 계층 ) |
| L4 ( 응용 계층 ) | L5 ( 세션 계층 ) + L6 ( 표현 계층 ) + L7 ( 응용 계층 ) |

# 프로토콜

: 통신 프로토콜 또는 통신 규약은 컴퓨터나 원거리 통신 장비 사이에서 메시지를 주고 받는 양식과 규칙의 체계

→ 즉 통신 규약 및 약속이다.

- 구문 : 전송하고자 하는 데이터의 형식, 부호화(Coding), 신호 레벨 등을 규정
- 의미 : 두 기기 간의 효율적이고 정확한 정보 전송을 위한 협조 사항과 오류 관리를 위한 제어 정보를 규정
- 시간 : 두 기기 간의 통신 속도, 메시지의 순서 제어 등을 규정

| OSI 계층 | 계층별 프로토콜 종류 |
| --- | --- |
| L1 ( 물리 계층 ) | - |
| L2 ( 데이터 링크 계층 ) | Ethernet, Token Ring, FDDI, Apple Talk |
| L3 ( 네트워크 계층 ) | IP, IPX |
| L4 ( 전송 계층 ) | TCP, UDP, SPX |
| L5 ( 세션 계층 ) | NetBIOS, SAP, SDP, NWLink |
| L6 ( 표현 계층 ) | ASCII, MPEG, JPEG, MIDI |
| L7 ( 응용 계층 ) | HTTP, SMTP, FTP, Telnet |

# HTTP, HTTPS

> 응용 계층 ( application layer )
> 
- HTTP : 데이터를 주고 받기 위한 프로토콜
    - Stateless : 상태 정보를 저장하지 않는다
    - Connectionless : 클라이언트의 요청에 맞는 응답을 보내고 연결을 끊는다
    - 장점
        - 통신간의 연결 상태 처리나 상태 정보를 관리할 필요가 없어 서버 디자인이 간단함
    - 단점
        - stateless 라는 특징으로 인해 이전 통신 정보를 모르기 때문에 매번 인증이 필요함
        - 이를 해결하기 위해 **쿠키**, **세션**을 이용하여 데이터를 처리함
- HTTPS : HTTP보다 보안이 강화된 프로토콜
    - HTTP는 평문 데이터를 전송 → 중요한 정보를 주고 받으면 제 3자가 조회 가능
    
    → 이 문제를 해결하기 위해 HTTP + 암호화 == HTTPS
    
    - SSL의 껍질을 쓴 HTTP
        - SSL (Secure Socket Layer) : 인터넷을 통해 전달되는 정보를 보호하기 위해 개발한 통신 규약
        
- HTTP는 TCP와 직접 통신
- HTTPS는 SSL과 통신 / SSL이 TCP와 통신 → 암호화, 증명성, 안전성 보호 가능

# TCP, UDP

> 전송 계층 ( transport layer )
> 
- TCP : 연결지향형 전송규약
    - 흐름 중심 프로토콜, 통신을 주고받는 것을 중요시함
    - 중간에 패킷이 손실되는 경우 재전송을 통해(SYN-ACK handshaking) 신뢰성을 보장함(느림)
    - 대부분의 통신에서 사용됨, 특히 파일이나 데이터 전송 시에 사용
    - 데이터 경계 구분이 없음 (바이트 스트림 서비스)
- UDP : 비연결지향형 전송규약
    - 데이터 중심 프로토콜, 주고받는 통신보다 데이터를 일방적으로 보내는 것을 중요시함
    - 데이터 전송의 신뢰성 보장 X, (빠름)
    - P2P, 스트리밍, 전화에 사용

# 전체적인 웹 동작 방식
 <img src="https://img1.daumcdn.net/thumb/R1600x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FblP6Co%2FbtruDDt3Rwm%2FAAAAAAAAAAAAAAAAAAAAAFYWA9Sc0PMUYmf914xrsdP309NM37MgAxkvPdh0k984%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1764514799%26allow_ip%3D%26allow_referer%3D%26signature%3D80ssdmX6CX0ZO9oxlMxMKU6ifT8%253D">

- 사용자가 브라우저에 URL([www.naver.com](http://www.naver.com/))을 입력
- DNS 서버에 도메인 네임으로 서버의 진짜 주소를 찾음
- IP 주소로 웹 서버에 TCP 3 handshake로 연결 수립
- 클라이언트는 웹 서버로 HTTP 요청 메시지를 보냄
- 웹 서버는 HTTP 응답 메시지를 보냄
- 도착한 HTTP 응답 메세지는 웹 페이지 데이터로 변환되고, 웹 브라우저에 의해 출력

# 💡중요 키워드 정리
| OSI 계층 | 중요 키워드 |
| --- | --- |
| L3 ( 네트워크 계층 ) | 패킷, 라우터 ( 라우팅 ), IP주소, NAT, SDN, DHCP, ISP, Firewall, 도커 |
| L4 ( 전송 계층 ) | TCP, UDP, 신뢰성 제공 방식 ( 흐름 제어, 혼잡 제어 … ), End-to-End, IP의 한계, L4 로드 밸런싱 |
| L7 ( 응용 계층 ) | HTTP(s),쿠키, 세션, 캐시, 세션 기반 인증 vs 토큰 기반 인증 ( JWT 토큰 ), CORS, API Gateway, L7 로드밸런싱 ( vs L4 로드밸런싱 , 쿠버네티스 ), DNS |
