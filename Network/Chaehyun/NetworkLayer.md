## 네트워크 계층

- **호스트↔호스트 간 전달 담당 계층**
  - 송신 측 트랜스포트 계층 세그먼트 → **IP 데이터그램(datagram)** 으로 캡슐화
  - 목적지 호스트에서 데이터그램 → 세그먼트로 디캡슐화 후 트랜스포트 계층으로 전달
- **라우터를 거치며 패킷을 전달하는 데 필요한 공통 규칙(IP, 라우팅 등) 제공**
- **Data Plane**
  - 한 라우터 내부에서 수행되는 작업
  - 도착한 패킷의 목적지 IP를 보고 **포워딩(Forwarding) 테이블** 기반으로 output 포트 선택
  - 고속 하드웨어(Switch fabric, ASIC, TCAM 등) 중심 처리
- **Control Plane**
  - 전체 네트워크 관점에서 **라우팅(Routing) 경로를 계산**하는 계층
  - 라우팅 프로토콜(LS, DV, OSPF, BGP 등)이 경로 계산
  - 계산 결과를 각 라우터의 포워딩 테이블로 뿌림

### SDN (**Software-Defined Networking)**

- Control plane을 네트워크 중앙의 **SDN 컨트롤러**로 모으는 구조
- 컨트롤러가 전체 네트워크 토폴로지·정책를 보고 경로 계산
- 각 라우터(스위치)는 컨트롤러가 내려준 **포워딩 규칙만 실행하는 data plane 장비** 역할

## 라우터 내부 구조와 Queueing

### 라우터 구성 요소

- **Input 포트**
  - 프레임 수신, 링크 계층 처리, 목적지 IP 파싱
  - 포워딩 테이블 조회 → 어느 output 포트로 보낼지 결정
- **Switch Fabric**
  - input 포트에서 output 포트로 실제 데이터그램 전달
  - 버스 / 크로스바 / 고속 네트워크 등 다양한 구조
- **Output 포트**
  - 큐(버퍼)에 전송 대기 패킷 저장
  - 스케줄링 알고리즘에 따라 링크로 전송

### Queueing과 HOL Blocking

- **Queueing 발생 지점**
  - **Input 포트 Queue**
    - Switch fabric 속도가 입력 속도보다 느린 경우 큐가 쌓임
    - **HOL(Head-of-Line) Blocking**
      - 큐 맨 앞 패킷이 특정 output 포트를 기다리느라 막혀 있을 때
      - 뒤에 있는 패킷이 다른 output 포트로 지금 당장 나갈 수 있어도 앞 패킷 때문에 함께 대기하는 현상
        <img width="1132" height="368" alt="image" src="https://github.com/user-attachments/assets/3d545680-84d8-4777-9e6b-d355f33ad67c" />

  - **Output 포트 Queue**
    - Switching 속도 > 링크 속도인 경우, 링크 대역폭 한계로 output 포트에서 큐 증가
      <img width="1262" height="369" alt="image" src="https://github.com/user-attachments/assets/b3ddc2e7-565c-4690-8f23-d968ec079de9" />

- **결과**
  - **Queueing delay 증가**
  - 버퍼 오버플로 시 **패킷 드롭(data loss)** 발생

### 패킷 스케줄링 알고리즘

- **FCFS (First Come, First Served)**
  - 도착 순서대로 전송
- **Priority Queueing**
  - high-priority / low-priority 큐 분리
  - high-priority 큐부터 비움
  - low-priority 패킷이 영원히 못 나가는 **Starvation(기아 상태)** 위험
- **Round Robin (RR)**
  - 여러 큐를 순환하면서 일정 시간 단위로 패킷 전송
- **Weighted Fair Queueing (WFQ)**
  - RR 변형
  - 각 큐마다 가중치 부여, 가중치에 비례한 전송 시간 배정
  - 서비스 품질(QoS) 위한 공정한 대역폭 분배 목적

## IP 데이터그램과 Fragmentation

### IP 데이터그램

- **트랜스포트 계층 세그먼트**를 감싸는 네트워크 계층의 패킷 단위
- 구조
  - **헤더 + 데이터**
  - 데이터: TCP/UDP 세그먼트 등 상위 계층 payload
- 오버헤드 예시
  - TCP 헤더 20B + IP 헤더 20B → **기본 40B 오버헤드**

### IP 헤더 주요 필드
<img width="1133" height="706" alt="image" src="https://github.com/user-attachments/assets/17e43e29-72b5-4cff-83dc-c8772c7d8a0b" />


- 버전(IPv4/IPv6), 헤더 길이
- 전체 길이(total length)
- 식별자(ID), flags, fragment offset
- TTL(Time To Live)
- 상위 계층 프로토콜(TCP/UDP 등)
- 헤더 체크섬
- 출발지 IP, 목적지 IP

### Fragmentation과 Reassembly

- **MTU(Maximum Transfer Unit)**
  - 링크 계층이 한 번에 보낼 수 있는 최대 프레임 크기
- 데이터그램이 MTU보다 크면
  - 라우터가 데이터그램을 **여러 조각으로 분할(fragment)**
  - 각 fragment는 **자기만의 IP 헤더** 보유
  - 목적지에서 **재조립(reassembly)** 수행
- ex)
  <img width="1266" height="636" alt="image" src="https://github.com/user-attachments/assets/1c78c632-64b5-4ac5-aeea-25a90bb96672" />

  - 원본 데이터그램 길이: 4000B (헤더 20B + 데이터 3980B)
  - MTU = 1500B → 데이터 부분 최대 1480B씩 송신 가능
  - 분할
    - 1번 fragment: data 1480B + header 20B = 1500B
    - 2번 fragment: data 1480B + header 20B = 1500B
    - 3번 fragment: data 1020B + header 20B = 1040B
  - 각 fragment가 **같은 ID**를 갖고, fragment flag / offset으로 순서와 마지막 여부 표시 → 재조립할 때 사용

## IP 주소 체계와 포워딩

### 계층적 IP 주소 구조

- IP 주소: **네트워크 부분 + 호스트 부분**
- 인접한 호스트일수록 **왼쪽 비트(상위 비트)가 유사**
  - 같은 서브넷 내 호스트들: 네트워크 부분 동일
  - 이 구조 덕분에 라우터가 **주소 범위(prefix)** 단위로 포워딩 테이블 구성 가능

### 서브넷(Subnet)

- 정의
  - 라우터를 거치지 않고 도달 가능한 **isolated 네트워크**
  - 실질적으로 하나의 **LAN** 과 같은 단위
- 표기: CIDR
  - ex)
    <img width="320" height="100" alt="image" src="https://github.com/user-attachments/assets/8ae9fe0b-6749-438b-8bb5-57a411fb46ba" />

    - `200.23.16.0/23`
    - 앞 23bit: 서브넷 부분
    - 나머지 9bit: 호스트 부분 → 최대 2⁹ = 512 호스트

### Longest Prefix Matching

- 라우터 포워딩 테이블 항목
  - `(prefix, mask) → output interface`
- 하나의 목적지 IP가 **여러 prefix** 와 매칭될 수 있음
- 이때 **“가장 긴 prefix(가장 많은 비트가 일치하는 항목)”** 를 선택
  - 더 구체적인 네트워크로 라우팅되는 효과
- 효율적 포워딩을 위해 TCAM 등 하드웨어에도 특화된 구조 사용
- ex)
  <img width="1098" height="342" alt="image" src="https://github.com/user-attachments/assets/6ac4d869-1153-4938-80b9-7d753a0133c7" />

  11001000 00010111 00010110 10100001는 0에, 11001000 00010111 00011000 10101010은 1에 매칭

## DHCP 주소 자동 할당 프로토콜

- 호스트가 네트워크에 접속할 때 **IP 주소·기본 게이트웨이·DNS 주소** 등을 **동적으로 임대(lease)** 받게 하는 프로토콜
- **Plug-and-play 네트워킹** 구현

<img width="916" height="511" alt="image" src="https://github.com/user-attachments/assets/cb8cf527-b3ac-4ae4-ba38-7d28742a6ad9" />


1. **DHCP Discover**
   - 호스트가 브로드캐스트로 “DHCP 서버 어디 있냐” 탐색
2. **DHCP Offer**
   - DHCP 서버가 사용 가능한 IP 주소 후보, lease 시간 등 제안
3. **DHCP Request**
   - 클라이언트가 선택한 서버 및 IP주소를 명시해 요청
4. **DHCP ACK**
   - 서버가 최종 할당 승인
   - IP 주소, 서브넷 마스크, 기본 게이트웨이, DNS 서버 주소 등 전달
5. 필요시 **lease 갱신 요청** 과정 반복

## NAT (Network Address Translation)

- **IPv4 주소 고갈 문제 완화**
  - 공인 IPv4 주소 32bit → 전 세계 모든 기기를 커버하기에 부족
- **주소 공유**
  - 하나의 공인 IP 주소 뒤에 **여러 사설 IP 호스트** 숨기기
- **보안 측면 이점**
  - 외부에서 내부 호스트 실제 IP를 직접 알 수 없음
  - 기본적으로 외부에서 임의 연결 열기 어려움

### NAT 동작 방식

- NAT 라우터 안에 **NAT Translation Table** 존재
  - `(내부 IP, 내부 포트) ↔ (공인 IP, 외부에서 보이는 포트)` 매핑
- 모든 **외부로 나가는 데이터그램의 출발지 주소**:
  - IP: NAT 라우터의 공인 주소로 변경
  - 포트: 고유 포트 번호로 재할당
- 들어오는 데이터그램:
  - (공인 IP, 목적지 포트)를 보고 테이블 lookup → 해당 내부 호스트로 전달
- 포트 번호 16bit → 하나의 공인 IP만으로 최대 **6만여 개 연결 세션** 관리 가능

### 단점

- **라우터가 포트 번호(전송 계층)를 들여다봄**
  - “네트워크 계층 장비는 IP 헤더까지만 봐야 한다”는 전통적 레이어링 원칙 위배
- **end-to-end 원칙 위반**
  - 중간 노드가 패킷 내용을 수정
- IPv6 보급 시 **근본적으로 불필요해지는 기능**이라는 시각

## IPv6

- **128bit 주소 공간**
  - 사실상 고갈 걱정 없는 수준
- **헤더 크기**
  - 고정 길이 40B 헤더
- **중요 변화**
  - 라우터에서 **fragmentation 미지원**
    - 단편화는 송·수신 호스트의 책임
  - **헤더 checksum 삭제**
    - 라우터 처리 속도 향상 (링크 계층, 트랜스포트 계층에서 이미 오류 검출)
  - options는 **extension header** 형태로 분리
  - **ICMPv6** 도입: IPv6 전용 제어 메시지, “Packet Too Big” 등 새로운 타입 추가

### IPv4 ↔ IPv6 : Tunneling

- 모든 라우터를 한 번에 IPv6로 교체 불가
- **터널링(tunneling)**
  - IPv6 데이터그램을 **IPv4 데이터그램의 payload** 로 캡슐화
  - IPv4만 이해하는 라우터 구간은 IPv4 헤더만 보고 전달
  - 터널 끝에서 다시 IPv6 데이터그램 꺼내서 처리

## Routing Control Plane

- 출발지 호스트 → 여러 라우터 → 목적지 호스트까지 가는 **“좋은 경로”** 결정
  - 좋은 경로 기준
    - 최소 비용(지연, 혼잡, 링크 비용 등)
    - 정책 고려(특정 ISP 피하기 등)
- **Static vs Dynamic**
  - Static: 라우팅 정보 거의 변하지 않음, 수동 설정
  - Dynamic: 링크 상태/비용 변화에 따라 자동으로 갱신
- **Global vs Decentralized**
  - Global (Link State, LS)
    - 각 라우터가 네트워크 전체 토폴로지와 링크 비용을 알고 있음
  - Decentralized (Distance Vector, DV)
    - 각 라우터는 **이웃 노드 정보**만 알고, 반복적인 메시지 교환으로 최단 경로 수렴

## Link State (LS)

- 각 라우터가
  - 자신의 이웃 링크 비용을 브로드캐스트(Link State Broadcast)
  - 모든 라우터의 링크 정보 수집 → **전체 그래프 G=(V,E)** 재구성
- 이후 한 노드(자기 자신)를 출발점으로 **Dijkstra 알고리즘** 수행
  - 모든 목적지까지의 최소 비용 경로 계산
  - 결과로 **포워딩 테이블** 생성

### Dijkstra 알고리즘

- 아이디어
  - 출발 노드 u에서 시작
  - **확정된 집합 N’** 과 **잠정 거리 D(v)** 관리
  - 매 단계에서 N’ 밖에 있는 노드 중 D(w)가 최소인 w 선택 → N’에 추가
  - w를 통해 이웃 노드 비용 갱신
    `D(v) = min(D(v), D(w) + c(w, v))`
- 복잡도
  - 단순 구현: O(n²)
  - 우선순위 큐 사용 시 O(n log n)

### Oscillations Possible Problem

- 링크 비용이 **혼잡에 따라 동적으로 변하는 경우**
  - 최단 경로 재계산 → 트래픽이 새 경로로 몰림 → 그 링크 혼잡 증가 → 비용 증가
  - 다시 경로 변경 … 이런 식으로 **경로가 계속 진동(oscillation)** 할 수 있음

## Distance Vector

- 각 노드 x가 유지하는 정보
  - 이웃 v까지의 링크 비용 `c(x, v)`
  - 각 이웃 v가 알고 있는 **distance vector** `D_v(y)` (v에서 y까지 최소 비용 추정)
- Bellman-Ford 수식
  - `D_x(y) = min_v { c(x, v) + D_v(y) }`
- 특징
  - **Iterative**: 반복적으로 값 갱신
  - **Asynchronous**: 업데이트 타이밍이 노드마다 다름
  - **Distributed**: 중앙 컨트롤러 없이 이웃과 메시지 교환만으로 수렴

1. 각 노드는 이웃 링크 비용과 자신의 distance vector를 초기화
2. 이웃으로부터 새로운 distance vector 수신 시
   - Bellman-Ford 수식으로 자신의 D_x(y) 재계산
3. 값이 변하면 다시 이웃에게 자신의 distance vector 전송
4. 반복 → 네트워크 전체가 동일한 최소 비용 값으로 **수렴(convergence)**

### Count-to-Infinity Problem

- 링크 비용이 커지거나 링크가 끊긴 상황에서
  - 잘못된 낮은 비용 정보가 이웃 간에 계속 돌면서
  - 실제보다 훨씬 큰 숫자로 천천히 증가하는 현상
  - 수렴까지 **매우 많은 iteration** 필요
- 해결 기법
  - **Poisoned Reverse**
    - 만약 노드 z가 y를 통해 x로 가는 경로를 사용한다면
    - z는 y에게 “나에서 x까지 비용 = ∞” 라고 광고
  - **Split Horizon**
    - 어떤 이웃으로부터 배운 경로 정보는 **그 이웃에게 다시 광고하지 않음**
  - **Triggered Update**
    - 정기적인 주기 업데이트를 기다리지 않고, **변화 발생 즉시** 업데이트 전송

## LS vs DV

- 메시지 복잡도
  - LS: 전체 토폴로지 정보 전송 → O(nE)
  - DV: 이웃과만 교환 → 상대적으로 적지만 convergence 시간 다양
- 수렴 속도
  - LS: 알고리즘 자체는 빠르지만 oscillation 가능
  - DV: count-to-infinity로 인한 매우 느린 수렴 가능
- 견고성
  - LS: 한 노드가 잘못된 링크 비용만 광고 → 나머지는 그 값만 사용
  - DV: 잘못된 distance vector가 여러 번 전파되며 전체 네트워크 결과 오염 가능

## 인터넷 라우팅 구조

### AS(Autonomous System)와 계층적 라우팅

- **AS**
  - 하나의 관리 주체(ISP, 대형 기업 등)가 관리하는 라우팅 도메인
  - 고유한 **ASN(AS Number)** 부여
- 인터넷 구조
  - 수많은 AS들이 서로 연결된 **네트워크들의 네트워크**
  - 한 AS 내부 라우팅: **Intra-AS**
  - AS 간 라우팅: **Inter-AS**
- 왜 계층적 구조가 필요한지
  - 전 세계 수억 개 목적지 주소를 단일 평면 라우팅으로 관리 불가
  - 각 AS가 **자기 내부는 자유롭게**, 외부와는 최소한의 정보만 교환하도록 설계

### Intra-AS 라우팅 – OSPF

- **OSPF (Open Shortest Path First)**
  - 대표적인 **link state 기반 Intra-AS 라우팅 프로토콜**
  - AS 내부 라우터가 서로 링크 상태 브로드캐스트 → 전체 토폴로지 학습
  - 각 라우터가 Dijkstra 알고리즘으로 **게이트웨이 등 모든 노드로의 최단 경로 계산**
- 대규모 AS에서의 구조
  - AS를 다시 여러 **area** 로 분할
  - 각 area 내부는 link state 수행
  - 경계 라우터(boundary router)가 area 간 요약 정보 전달
  - 장점
    - 라우팅 테이블 크기 감소
    - 링크 변화 시 업데이트 트래픽 감소

### Inter-AS 라우팅 – BGP

- **BGP (Border Gateway Protocol)**
  - 현재 인터넷의 사실상 표준 **Inter-AS 라우팅 프로토콜**
  - **Policy-based** 라우팅
    - 단순 최단 경로가 아니라 각 AS 정책(비용, 계약 관계 등)을 반영
- 두 가지 형태
  - **eBGP** (External BGP)
    - 서로 다른 AS의 gateway 라우터 사이에서 실행
    - “우리 AS에서 도달 가능한 prefix” 정보 교환
  - **iBGP** (Internal BGP)
    - 한 AS 내부 라우터들 사이에서 eBGP로 받은 정보 전파
- BGP가 전파하는 정보
  - **Prefix**: 도달 가능한 목적지 주소 블록
  - **Attributes**
    - **AS-PATH**: 어떤 AS들을 거쳐 도달하는지 리스트
    - **NEXT-HOP**: 다음 AS로 넘어갈 gateway 라우터 주소
- 경로 선택 정책 예시
  - **Shortest AS-PATH**
    - 경로 상의 AS hop 수가 작은 경로 선호
  - **Hot-Potato Routing**
    - 내부 비용을 최소화하려고 **가장 가까운 NEXT-HOP** 으로 빨리 트래픽 밀어내기
