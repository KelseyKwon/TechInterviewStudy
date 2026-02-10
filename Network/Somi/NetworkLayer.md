# 1. 네트워크 계층 ( L3 )

: 경로 ( 루트 ) 와 주소 ( IP ) 를 정하고 패킷을 전달

- 단위 : 패킷 ( Packet/ Datagram )
    
    <img src="https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcRMdA8s1rUNDMmaiZ2YewU-Dse8ofAoiNohH2E1yvsqHuzjqNed">
    

목적지까지 가장 안전하고 빠르게 데이터를 보내는 기능 = 최적의 경로를 설정해야함

→ 이러한 **라우팅 기능**을 맡은 것이 네트워크 계층 !

# 2. 패킷

pack+bucket : 컴퓨터 간 네트워크를 통해 데이터를 주고받을 때 전송되는 데이터 조각 )

- 큰 데이터를 작은 조작으로 나누어 전송 → 목적지에서 패킷 재조립
    - 데이터가 클 경우 네트워크의 대역폭을 너무 많이 차지하여 다른 패킷의 흐름을 막음 → 병목현상 발생 !
        
        ---
        
        💡 대역폭 ( bandwidth ) : 일정 시간 내 네트워크 연결을 통해 처리할 수 있는 데이터 양 ( 네트워크가 데이터를 얼마나 빨리 전송할 수 있는지 판단 )
        
        ---
        
- 중간에 문제 발생할 경우 손상된 패킷만 재전송 → 안정적인 데이터 통신

# 3. 주요 프로토콜 : IP

### IP ( Internet Protocol )

: 네트워크 계층에서 통신하는 주요 프로토콜 , 패킷의 전달을 책임진다

> **⁉️ IP는 신뢰성 없는 비연결형 전송을 제공함**
> 
> 
> 데이터를 패킷으로 보내는 역할에만 충실, 데이터를 어떻게 보낼지 관여 X
> 
> - 비연결성 : 패킷을 받을 서버가 없거나 서비스 불능이어도 모른다
> - 비신뢰성 : 패킷이 분실, 손상될 수 있고 순서가 바뀔 수 있음
> 
> ✅ 이 때 데이터를 유실하지 않고 전달하는 프로토콜이 전송계층의 TCP 인것
> 

### IP 구조

<img src="https://velog.velcdn.com/cloudflare/redgem92/1002b979-a301-48e9-b88a-8a97239084f0/image.png">

- Data Packet == Frame ( 데이터 링크 계층 ) == Header + Payload
- Header : 목적지, 출발지 IP 주소 등등
- Payload : 전송되는 데이터

### IPv4 헤더 구조

<img src="https://velog.velcdn.com/cloudflare/redgem92/6f15fc44-abc1-4563-82c5-0047e03522d2/image.png">

1. Version : IP의 버전, IPv4

2. Header Length : 헤더의 길이 ( 4 바이트 단위로 → 520 ~ 1560 바이트 )

3. Type of Service : 서비스 품질

4. Total Packet Length : IP 패킷의 전체 길이를 바이트 단위로 → 최대 65535

5. Identifier, Flag, Offset : IP Fragment 필드를 단편화와 재조합하는데 사용되는 값 → 한번에 보내기 큰 패킷을 작은 단위로 전송할 때 사용

6. Time to Live ( TTL ) : IP 패킷의 수명

7. Protocol ID : 데이터에 포함되는 상위 계층의 프로토콜 정보 ( TCP,UDP )

8. Header Checksum : 오류 검출 코드

9. Source/Destination IP Address : 출발, 목적지 IP 주소

10. IP Hedaer Options & Padding : 옵션 ( 테스트 및 디버깅 )

### IP 주소

: 컴퓨터간 통신을 하기 위해 사용되는 네트워크 상 주소

( 0~255 : 0~255 : 0~255 : 0~255 ) == 옥텟 4개로 구성

→ 옥텟 ( Octet ) : IP 주소 하나의 공간 단위

> **⁉️ IPv4 vs IPv6**
> 
> 
> ✅ 주요 차이점은 주소 길이와 형식
> 
> IPv4 는 32비트 주소를 사용하므로 2^32 = 약 43억개의 고유 주소 제공함
> 
> → 4개의 10진수로 표현 ( 0~255 : 0~255 : 0~255 : 0~255 )
> 
> IPv6 은 IPv4의 주소 부족 문제를 해결하기위해 개발되었음
> 
> → 128비트 주소를 사용하며 거의 무한에 가까운 주소 공간을 제공함
> 
- 서브넷 마스크 ( Subnet Mask )
    
    : IP 주소를 네트워크 부분과 호스트 부분으로 구분하는 데 사용
    
    → IP 주소 공간을 효율적으로 관리하고, 네트워크 성능과 보안을 향상시킬 수 있음
    
    EX ) 255.255.255.0  → 세 옥텟이 네트워크 식별 / 마지막 옥텟이 호스트 식별
    
    > **⁉️ IP주소 = ( 네트워크 부분 + 호스트 부분 )**
    > 
    > 
    > <img src="https://encrypted-tbn1.gstatic.com/images?q=tbn:ANd9GcRt9Vq6kl7keHAIcuEyHJ8Qenqp6310VRZPjOGd3dWX8QUi8YR5">
    > 
    > 
    > EX ) 철수, 민수, 영희가 수원에서 만나기로 함
    > 
    > 철수와 민수는 같은 수원시에 살아서 시내버스 이용
    > 
    > 영희는 안양에 살아서 시외버스 이용
    > 
    > ---
    > 
    > 수원, 안양, 용인 ( 지역 ) = 네트워크 ( 브로드캐스트 영역 ) = 라우터
    > 
    > 철수, 민수, 영희 ( 사람 ) = 호스트 ( 개별 단말기 영역 ) = PC
    > 
    > 시내 = 브로드캐스트 스위칭  /    시외 = 라우팅
    > 
    > ---
    > 

> **IP 주소 vs 포트**
> 
> - IP 주소 : 건물 주소
> - 포트 : 건물 안의 특정 호실
> 
> **⁉️ 만약 포트가 안 열려있다면 ?!**
> 
> : 서비스에 연결할 수 없음, 통신 불가
> 
> ```jsx
> [ 인터넷 ] 
>       ↓
>   IP 주소(집 주소)
>       ↓
>  ┌────────────────────────┐
>  │ 컴퓨터 내부 (방 여러 개)    |
>  │                        │
>  │ Port 80  → Web Server  │
>  │ Port 22  → SSH Server  │
>  │ Port 443 → HTTPS       │
>  │ Port 3306 → Database   │
>  └────────────────────────┘
> ```
> 

# 3+) 주요 프로토콜 : ICMP, ARP, RARP

### ICMP ( Internet Control Message Protocol )

인터넷 제어 메시지

: 네트워크 진단과 오류 보고에 사용됨

- IP의 단점 ( 비신뢰성, 비연결성 ) 을 보완하고자 사용되는 IP supported 프로토콜
    
    → IP 헤더가 할 수 없는 통신 오류 찾아주는 등의 일을 하기 위해 만들어짐
    
    data를 보내려는 목적이 아니라, 통신이 잘 이루어지게 하는 것
    
    > **⁉️ IP의 단점 ( 비신뢰성, 비연결성 )**
    > 
    > 
    > 보완하는 것 → TCP 
    > 
    > 오류가 일어나면 알려주는 것 → ICMP 
    > 
- L3의 프로토콜이지만 IP보다 상위에서 동작함

### ARP ( Adress Resolution Protocol ) ↔ RARP ( Reverse ARP )

:  ( 논리적 ) IP 주소를 ( 물리적 ) MAC 주소로 변환하는 역할 ↔ MAC 주소를 IP 주소로

- L3의 프로토콜이지만 IP보다 하위에서 동작함
- 목적지 MAC 주소를 확인할 때 주로 쓰임

### +L7 )DHCP ( Dynamic Host Configuration Protocol )

: **응용 계층**의 프로토콜

: 네트워크 장비들에게 동적으로 IP 주소, 네트워크 설정 정보를 자동으로 할당

( → IP 주소, 서브넷 마스크, 게이트웨이, DNS 서버 정보 등 제공 )

> **⁉️ DHCP가 RARP를 대체함**
> 
> 
> RARP는 IP 주소 할당에 국한되어 있었음
> 
> → 정적인 IP 할당 방식
> 
> DHCP는 네트워크 설정 관리 등이 훨씬 유연하고 효율적으로 이루어짐
> 
> → DHCP는 IP 주소 임대 방식으로 네트워크에 연결된 장치들이 동적으로 IP 주소를 할당 받게 함, 필요할 때 갱신
> 

# 4. 라우터 ( vs 스위치 )

: 네트워크 간 패킷을 전송해주는 장비

- 라우팅 : 을 이용해 적절한 경로를 선택시킬 수 있음
- 패킷에서 목적지 IP 주소를 확인하고 하나 이상의 네트워크 간의 패킷의 경로를 선택하여 전송

- 라우팅 프로토콜
    
    : 패킷이 목적지까지 효율적으로 전달되도록 경로를 설정하는 방법을 정의하는 프로토콜
    
    <img src="https://velog.velcdn.com/cloudflare/redgem92/dd1c3706-8b14-4a41-b0a1-a3ef08fac9b5/image.png">
        
    - 정적 라우팅 : 기본적인 라우팅 방식, 수동으로 경로를 라우터에 설정
        - 라우팅 테이블에 목적지 IP주소, 라우터를 의미하는 인터페이스 정보 설정
    - 동적 라우팅 : 네트워크 상황에 따라 최적의 경로로 변경됨
        - AS ( Autonomous System ) : 하나의 라우터 집단, 단체를 구분하는 단위
        - EGP : AS 간 연결되는 부분
            - BGP ( Border Gateway Protocol ) : 자율 시스템 간의 경로 설정
        - IGP : AS 내 프로토콜 ( RIP , OPSF )
            - RIP ( Routing Information Protocol ) : 거리를 기준으로 경로 선택
            - OPSF ( Open Shortest Path First ) : 링크 상태 라우팅으로 최적의 경로 찾음
        
        > ⁉️ **SDN ( Software Defined Networking )**
        > 
        > 
        > : 네트워크 장비 ( 스위치, 라우터 ) 의 제어 기능을 중앙에서 소프트웨어로 관리
        > 
        > - 네트워크가 두가지로 나뉨 : Control Plane, Data Plane
        > - Control Plane
        >     - SDN 컨트롤러가 중앙에서 네트워크 정책, 라우팅 등을 결정함
        > - Data Plane
        >     - 스위치, 라우터가 지시받은대로 실제 패킷을 전달함
        
- 동적 라우팅 알고리즘
    - Distance Vector : 분산해서 업데이트하는 방식 ( 벨만 포드 기반 )
        - 인접 노드와 교환하는 방식으로 각 라우터들에 의해 최소 비용 경로를 계산
            - 목적지 IP까지의 거리 = hop count( 라우터간 거리+인터페이스 방향 )
        - 인접 라우터들과 주기적으로 라우팅 테이블을 교환, 확인하고 관리함
            
            → 인접 라우터들만 관리하기 때문에 메모리 절약, 구성이 간단하지만
            
            → 변경이 이뤄지지 않아도 주기적인 업데이트가 되어 무의미한 트래픽 발생!
            
    - Link State Louting : 중앙 집중형 업데이트 방식 ( 다익스트라 기반 )
        - 하나의 토폴로지 안의 정보를 통해 최소 비용 경로를 계산
        - 대규모 네트워크에 적합하며, 이벤트 기반의 라우팅 테이블을 관리함
            - 대역폭을 고려해서 경로마다 가중치를 부여, 네트워크 토폴로지 경로를 모든 라우터들에게 전달 ( 정보가 변경되는 이벤트일떄만 전파 )
                
                → 이벤트 건에서만 전파하기 때문에 트래픽이 감소함
                
                → 전체 네트워크의 라우팅 테이블 정보가 동일하게 유지됨
                

---

> **공인 IP vs 사설 IP**
> 
> 
> ✅ 
> 
> 공인 IP
> 
> - 인터넷 서비스 제공자 ( ISP ) 가 제공하는 IP 주소 ( ISP : KT,SK,LG같은 곳 )
> - 전세계에서 유일, 인터넷에서 누구나 직접 접근 가능함
> 
> 사설 IP ( Private IP )
> 
> - 라우터가 내부 기기들에게 배정함 **( DHCP 기능을 사용함 )**
> - 같은 네트워크 내부에서 사용하는 내부 IP주소
>     - 집/회사의 네트워크 내부에서 사용, 내부 전용 방번호
> 
> ```jsx
> 인터넷
>    │
>    │ (공인 IP: ISP가 제공)
>    ▼
> [ 라우터 ] + NAT 도 수행
>    │ DHCP로 사설 IP 자동 할당
>    ├── 192.168.1.2 (노트북)
>    ├── 192.168.1.3 (스마트폰)
>    └── 192.168.1.4 (TV)
> ```
> 

> **⁉️ 어떻게 공인 IP 하나로 여러 기기가 인터넷을 써?!**
> 
> 
> ✅ **NAT ( Network Address Translation )** 이 여기서 등장함 !
> 
> 집, 회사의 네트워크 내부 기기들은 사설 IP를 쓰기 때문에 전세계 기준으로 중복될 수 있다
> 
> → 하지만 인터넷 상에서는 유일한 공인 IP 가 필요함
> 
> → NAT가 사설 IP 를 공인 IP로 변환해서 외부로 내보내는 것 !!
> 
> - [내부 여러 기기] → 공인 IP 1개로 나가도록 변환하는 것
> - 내부 여러 기기를 **포트번호로** 누가 누구인지 구별함

---

### 라우터 vs 스위치

- 라우터 : 네트워크 간 데이터 통신을 담당
    - 서로 다른 네트워크(LAN ↔ 인터넷, LAN ↔ LAN)를 연결
- 스위치 : 네트워크 내부의 데이터 전송을 담당
    - 같은 네트워크(LAN) 안에서 기기끼리 연결
        
        LAN ( Local Area Network ) : 집, 회사 등에서 기기들이 연결된 네트워크
        

| 항목 | 스위치 | 라우터 |
| --- | --- | --- |
| OSI 계층 | L2 ( 데이터링크 ) | L3 ( 네트워크 ) |
| 기준 주소 | MAC 주소 기반 ( 집 호수 ) [ 집 호수를 보고 같은 아파트 내 배달 ] | IP 주소 기반 ( 건물, 도시, 도로 ) [ 외부 지역으로 배달 ] |
| 역할 | 같은 LAN 내부 패킷 전달 | 다른 네트워크 간 패킷 전달 |
| 예시 | 집 내부 기기 연결 ( PC-프린터 ) | 집 - 인터넷 연결 |

# 5. 방화벽 ( Firewall )

: 미리 정해진 보안 규칙에 따라 들어오고 나가는 네트워크 트래픽을 모니터링하고 제어하는 네트워크 보안 시스템

- 신뢰할 수 있는 네트워크 - 신뢰할 수 없는 네트워크 사이의 장벽 역할
    - 특정 트래픽을 허용할지 말지 결정함
- 종류
    - 패킷 필터링 : 네트워크 계층, 전송 계층
        - 패킷 자체만을 보고 미리 설정된 정책에 따라 허용, 거부 결정
        - 특정 IP, 포트를 허용, 거부 결정
        - 비용이 저렴하고 속도가 빠름, 사용자 인증은 어려움
    - 상태기반 패킷 검사 : 네트워크 계층
        - 패킷이 아닌 세션 단위의 검사
        - 네트워크 계층에서 패킷 필터링 및 TCP 연결에 관한 정보를 기록
    - 어플리케이션 게이트웨이 방식 : 응용 계층
        - 통과하는 패킷의 헤더 안의 데이터 영역까지도 체크함

### +) WAF ( Web Application Firewall )

: 리소스를 과도하게 사용하거나 보안을 위협하는 웹 공격으로부터 웹 앱이나 API를 보호

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fc1azuQ%2Fbtrg7YmcMrw%2FAAAAAAAAAAAAAAAAAAAAAAP1veiIOauWyDAVZNW0a8OW41EgwakTbafYPQgRAskq%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1767193199%26allow_ip%3D%26allow_referer%3D%26signature%3DHSUZtVpKBIU8SUM97a7UaVZ1Ykg%253D">

단순 방화벽은 → TCP/IP 레벨에 포함된 정보를 기반으로 차단 정책 생성

웹 방화벽 → HTTP 정보를 바탕으로 차단 정책 생성

# 6. 도커

: ‘컨테이너’ 라는 가벼운 가상환경을 만들고 실행하는 플랫폼

: 애플리케이션을 격리된 환경에서 실행하도록 도와줌

→ 서버, 컴퓨터 위에서 앱을 독립된 공간 ( 컨테이너 ) 에서 돌리는 기술

### 도커 컨테이너 / 도커 이미지

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FFXhcT%2FbtsGVKnxL79%2FAAAAAAAAAAAAAAAAAAAAANM9X55kbdCfH8ASk1FdXUHyI68H_nT-TP3zvZMq1PHE%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1767193199%26allow_ip%3D%26allow_referer%3D%26signature%3Dikfgnpd3XLR69gFb5%252Fga56k3S7s%253D">

- 도커 이미지 : 컨테이너를 만드는 데 사용되는 읽기 전용 템플릿
- 도커 컨테이너 : 도커 이미지라는 템플릿에서 생성되고 배포된 인스턴스

EX ) 도커 이미지 = 책을 출판하기 위한 원고 / 도커 컨테이너 = 책을 읽는 독자

책을 출판하기 위해서 내용이 있어야하는 것처럼 도커 이미지에 앱 실행 위한 모든 정보가 있음.

책을 출판하면 여러 독자들이 생기는 것처럼 도커 이미지를 실행하면 여러 도커 컨테이너가 실행되고 , 이 컨테이너 안에서 애플리케이션이 실행됨 ( 각 컨테이너는 독립적 )

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FlywGx%2FbtsGVTea2Ds%2FAAAAAAAAAAAAAAAAAAAAACNtnTFZHvTSUcaWHIZ8jeIwXKpfi0PFQW3z1k7e4vPz%2Fimg.jpg%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1767193199%26allow_ip%3D%26allow_referer%3D%26signature%3DXW5jFwJPcoI74a5yaQSUTueTPTs%253D">

- 특징
    - 컨테이너 생성, 실행
        - 컨테이너 = 작고 가벼운 가상 머신 같은 실행 환경
        - OS 전체를 복사하지 않고 **커널 ( 하드웨어 -프로그램의 인터페이스 ) 공유** → 가벼움, 빠름
    - 이미지 관리
        - 앱 실행에 필요한 모든 파일/라이브러리/환경을 담은 패키지
        - 어디서든 동일하게 실행 가능 (환경 차이 제거)
    - 격리
        - 컨테이너끼리는 서로 분리된 독립된 공간 사용 → 보안, 충돌 방지
    - 이식성
        - 같은 이미지면 어디서든 똑같이 실행됨 ( 개발 → 테스트 → 배포가 쉬워짐 )

### 도커 네트워크

: 도커 컨테이너는 서로 통신, 외부 인터넷과 통신, 호스트( 내 PC )와도 포트를 공유해야함

<img src="https://postfiles.pstatic.net/MjAyNDA1MDlfMTgg/MDAxNzE1MjM1MDQzMTIw.zHVndd_GlvOq5CFuYoEw6-rFBqMpPMEb83gxe7uVmvEg.WOdLPtHWclfpmO2RzqEirkQfu40vofykgWEnmTY6rZ8g.PNG/01.png?type=w773">

- **eth0  :** 실제 IP가 할당되는 host 네트워크 인터페이스
    - 가정에 제공되는 인터넷이 공유기를 통해 장비와 연결된다.
- **docker0 :** 도커가 설치될 때, 기본적으로 구성되는 브릿지
    - 호스트( 내 PC )와 컨테이너의 연결을 해주는 역할
    - 호스트의 제한된 IP를 여러 컨테이너에 공유하는 공유기의 역할
- **veth :** 컨테이너 내부와 연결해 주는 역할을 하는 가상 인터페이스
    - Container가 생성되는 동시에 veth가 생성되며, eth0과 연결되어 외부와의 통신이 가능

<aside>
💡

**결론 ‼️**

** -> 인터넷은 공유기를 통해 나의 맥북에 eth0으로 연결되고, eth0는 docker0에 연결되어 컨테이너와 연결된다**

</aside>

### 도커 네트워크 드라이버

- Bridge 네트워크 : 가장 기본적, 많이 사용되는 네트워크
    
    : 가상 네트워크 내부의 각 컨테이너는 네트워크 내에서 **고유한 IP 주소를 할당**받아 외부와의 통신을 허용
    
    - 동일한 Bridge 네트워크에 연결된 컨테이너는 통신할 수 있음
    - 동시에 해당 Bridge 네트워크에 연결되지 않은 컨테이너로부터는 격리함

→ Docker 시작 시 자동 설정, 별도 지정 없으면 기본 네트워크로 사용됨

# ⁉️ 예상 질문 모음

> **✅ 기술 면접 답변틀**
> 
> 1. 기술의 정의 
> 2. 어디에서 사용하는지
> 3. 어떻게 사용할 수 있는지

<aside>
💡

**네트워크 계층의 주요 역할은 무엇인가요? 이를 위해 사용하는 프로토콜은 어떤 것이 있나요?**

**IP 주소에는 IPv4와 IPv6가 있습니다. 이 두 가지의 차이점은 무엇인가요?**

**IP는 무엇인가요?**

**IP 주소와 포트는 어떠한 관계가 있나요?**

**만약 포트가 열려 있지 않으면 어떻게 되나요?**

**공인 IP와 사설 IP의 차이에 대해 설명하세요.**

**유니캐스트, 브로드캐스트, 멀티캐스트, 애니캐스트에 대해 설명하세요.**

**NAT(Network Address Translation)에서 IP와 포트는 어떻게 사용되나요?**

**서브넷 마스크는 무엇이며, 네트워크를 분할하는 데 어떻게 사용되나요?**

**라우팅 프로토콜이란 무엇이며, 대표적인 라우팅 프로토콜은 어떤 것들이 있나요?**

**라우팅 알고리즘에 대해 설명하세요.**

**라우터와 스위치의 차이에 대해 설명하세요.**

</aside>
