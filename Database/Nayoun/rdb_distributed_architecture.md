# Relational Database

> 데이터를 테이블 형태로 저장하고 관리하는 데이터베이스 모델
> 

***WHY RELATION?***

→ 여러 테이블로 분리된 데이터들이 서로 연결될 수 있기 때문에! 

- (`Primary Key`, `Foreign Key`을 사용해서 행을 연결한다)
- `Normalization`으로 테이블을 분리하고 중복 제거
    - 1:1, 1:N, N:N 관계가 존재함.

- 특징
    - SQL 사용 (CRUD 작업 수행)
    - 트랜잭션
        - 데이터베이스 내에서 수행되는 작업의 논리적 단위, 데이터의 무결성을 보장함!

# NoSQL

> `Not Only SQL` 
→ `RDBMS` + 비정형 데이터 처리를 위해 추가적인 특성을 지원하는 DB!
> 
- 특징
    - **Schema-less**
        - Schema를 미리 정의하지 않고 동적으로 속성을 수용할 수 있어 유연
    - **Scalability**
        - Clustering을 통해 성능을 수평적으로 확장하기 쉽다!
    - **고성능**
        - 단순 검색 및 추가 작업에 최적화된 키-값 저장 기법을 사용하여 응답 속도가 매우 빠르다.
    - **Transaction(ACID) 미보장**
        - 데이터의 엄격한 완결성보다는 **속도와 가용성**에 더 중점을 둔다.

| **종류** | **특징** | **대표 제품** |
| --- | --- | --- |
| **Key-Value DB** | 키와 값의 쌍으로 저장하는 가장 단순한 형태 | Redis, Riak |
| **Wide Columnar Store** | Column Family 모델을 사용하며 대용량 처리에 적합 | **Cassandra, HBase, ScyllaDB** |
| **Document DB** | JSON, XML과 같은 문서 구조로 데이터를 관리 | **MongoDB**, CouchDB |
| **Graph DB** | 데이터 간의 관계를 노드(Node)와 간선으로 표현 | Neo4J |

# Redis

> RAM 기반의 Key-Value 구조를 가진 Non-Relational 데이터 관리 시스템
> 
- In-memory 방식
    - 메모리에서 데이터 처리 → Read/Write 속도가 매우 빠르다
- Persistance
    - 메모리 → 휘발성이지만, Redis는 스냅샷이나 로그 파일 형태로 데이터를 디스크에 저장해 복구할 수 있는 기능을 제공.
- 다양한 자료구조
    - Strings, List, Set, Hashed, Sorted-Set 등 다양한 자료구조를 제공한다.

### 주요 활용 사례

- Look aside Cache
    - 데이터 찾을 때 먼저 캐시 확인 → 없으면 DB에 접근하고 결과를 다시 캐시에 저장
- Write back
    - 쓰기 요청이 몰릴 때 캐시에 모아 두었다가 일정 주기마다 DB에 batch 처리.
1. `Twitter`가 `caching`을 사용해서 타임라인 요청을 잘 처리함!
2. `Session Store` 
    1. 분산 서버 시스템에서 로그인 상태를 공유하기 위해 사용
        1. 독립적인 Redis Server을 Session Store로 활용해서 세션 불일치 문제를 해결.

### Redis 사용할 때 주의할점

- 메모리 할당 & 해제가 반복
    - 빈 공간이 생겨서 실제 사용량보다 더 많은 물리 메모리를 점유하게 된다. 이로 인해 프로세스가 강제 종료될 수 있으므로 여유 있게 관리해야 함!
- `string` : `key-value` 형태
- `List` : 링크드 리스트 자료구조
- `set` : 중복을 허용하지 않는 자료구조
- `sorted set(ZSet)` : `set` 자료구조에 `score`을 주고 이를 기준으로 순서 유지
- `hash` : `key` 내부에 `key value` 형태의 `Map` 구조
- Bit arrays, HyperLogLogs, Streams..

# Memcached

> `Multi-thread`를 지원하는 고성능 분산 메모리 `caching system`
> 
- `DB`나 `API` 호출 또는 렌더링 등으로부터 받아오는 결과 데이터를 작은 단위의 `key - value` 형태로 메모리에 저장한다.

**특징**

- `Consistent Hashing Library`을 통해 데이터를 분산한다.
- 데이터 구조가 Hash 형태 → 검색 시간 복잡도가 `O(1)` 빠르다!
- `Repcached Program`을 통해 `Master/Master Replication`이 가능 → 메모리 데이터 유실 시 즉시 복구가 가능!
- `slab` 할당자 사용해서 메모리 재할당 없이 관리, 트래픽이 몰려도 `Redis`에 비해 응답 속도가 안정적이고 메모리 파편화가 적다.
- `Redis`보다 `MetaData`를 적게 사용하고, 정적인 데이터를 캐싱할 때 내부 메모리 관리가 더 능률적이다.

**단점**

- 메모리 파편화 발생 가능

**성능**

<img width="250" height="459" alt="스크린샷 2026-02-27 오후 3 07 55" src="https://github.com/user-attachments/assets/9d9b3220-1206-498a-9e0d-51cc41a2676b" />

- **`without memcached`**
    - 각각의 웹 서버가 독립적인 캐시 공간을 가지고 있다 → 최대 캐시 크기가 64MB!
- **`with memcached`**
    - 각 서버의 가능한 메모리를 논리적으로 하나로 결합한다 → 두 서버의 64MB 공간이 합쳐져 → 128MB의 통합 캐시 공간을 사용할 수 있게 된다!
    

# Elastic Search

- `Elastic Stack` 구성
    - `ElasticSearch`를 중심으로 데이터를 수집, 전송, 시각화한다.
        - **`Beats`** : 경량 데이터 전송기.
        - **`Logstash`** : 다양한 플러그인을 이용한 데이터 수집 및 가공.
        - **`Elasticsearch`** : 분산형 검색 및 분석 엔진.
        - **`Kibana`** : 데이터 시각화 및 시스템 관리 UI 도구.

1. **`Search`**
    1. 인덱싱 및 분석
        1. HTML 태그 등 불필요한 글자를 없앤다.
        2. Tokenizer : 문장을 단어 단위로 자른다.
        3. Token Filter : 대문자 → 소문자로 바꾸거나, 불필요한 단어를 없앤다. 
    2. **`Sharding`**
        1. 거대한 데이터를 여러 개의 작은 조각으로 나누어 여러 컴퓨터에 분산해 저장한다.
    3. 통계 및 정렬
        1. 데이터를 한 줄씩 읽는 게 아니라, 열 단위로 모아서 저장한다. 
2. **`Observability`**
    1. 시계열 데이터 관리 - 시간의 흐름에 따라 데이터의 중요도가 달라지는 것을 관리한다.
        1. **`Hot`** : 방금 생긴 데이터 → 빠른 저장 장치에!
        2. **`Warm`** / **`Cold`** : 오래된 데이터는 느리지만 싼 저장장치로
        3. **`Delete`** : 너무 오래된 것은 자동으로 삭제한다. 
    2. **`Kibana`**
        1. Discover : 원본 데이터를 직접 검색.
        2. Visualize : 차트 or graph를 만든다. 
        3. DashBoard : 차트들을 한 화면에 모아 관리한다. 
3. **`Security`**
    1. 인증 및 권한
        1. 인증 : 회사에서 쓰는 공용 아이디 시스템과 연결해서 로그인을 관리.
        2. RBAC / ABAC : 직급이나 특정 속성에 따라 권한을 세밀하게 허락해준다. 
    2. 통신 보안
        1. 서버끼리 데이터를 주고받을 때 암호화(TLS / SSL)을 해서 보낸다.

### vs RDBMS %like%

- 성능
    - 인덱스 사용x (Full Table Scan)
        - 시간 복잡도는 O(N)이 된다.
    - 인덱스 활용 (Range Scan)
        - %를 접두사로 사용 x → 인덱스를 활용할 수 있어 성능이 나아짐! 하지만 여전히 Range Scan이 일어남.

<aside>
💡

약 17,725개의 데이터를 대상으로 수행한 테스트 결과는 다음과 같다.

- **MySQL `LIKE`**: 약 **0.013 ~ 0.014초**의 실행 시간이 소요
- **`ElasticSearch`**: 단순 키워드 쿼리 검색 시 약 **0.002초**가 소요
</aside>

# 데이터베이스 분산 기법

## 1. DB Clustering (서버 분산) → CPU / RAM을 분산

> 여러 대의 **DB server**을 묶어서 구성 → **하나의 저장소를 공유**한다.
> 

***서버 한 대가 죽어도 서비스가 유지된다! (고가용성)***

다른 서버가 역할을 대신해주기 때문에.

- `High Availability` (장애 발생 시 자동으로 다른 노드로 서비스 전환)
- `Scalability` (트래픽 증가 시 동적으로 노드 추가, 처리 성능 향상)
- `Load Balancing` (여러 노드에 요청을 분산 → 성능 최적화!)

<img width="562" height="218" alt="스크린샷 2026-02-27 오후 3 08 22" src="https://github.com/user-attachments/assets/6a14713f-a5d6-4835-a280-5f77d6156345" />


| **구분** | **`Active-Active` (Active Clustering)** | **`Active-Standby` (Standby Clustering)** |
| --- | --- | --- |
| **운영 상태** | 모든 DB 서버를 동시에 **`Active`** 상태로 운영 | 한 대는 **`Active`**, 나머지는 **`Standby`** 상태로 대기 |
| **장애 대응** | 한 서버가 죽어도 다른 서버가 즉시 가동되어 **중단 없음** | 장애 발생 시 `Standby` 서버를 Active로 **전환하는 시간** 필요 |
| **자원 효율** | 모든 서버의 **CPU, 메모리 이용률**을 극대화함 | 평상시에는 한 대의 서버 자원만 활용함 |
| **비용/성능** | 여러 대 운영으로 **비용 높음**, 공유 저장소 **병목** 가능성 | Active-Active 방식 대비 **상대적으로 저렴**한 비용 |

## 2. Replication (부하 분산) → Traffic을 분산

> 데이터를 복사해서 별도의 저장소를 가진 복제본들을 유지한다.
> 

> 여러 개의 `DB`를 권한에 따라 수직적인 구조 (`Master-Slave`)로 구축하는 방식!
> 

여러 서버가 읽기 요청을 나누어 처리할 수 있게 한다. 

<img width="238" height="151" alt="스크린샷 2026-02-27 오후 3 08 35" src="https://github.com/user-attachments/assets/f7d52d38-076f-4e25-be78-8be60d849bcc" />


- `Master node`
    - Write, Update, Delete 작업을 처리한다.
- `Slave node`
    - `Read` 작업을 처리한다.

### 데이터 처리 과정

1. `Master` 노드에 쓰기 트랜잭션이 수행된다.
2. `Master` 노드는 데이터를 저장하고 트랜잭션에 대한 로그를 파일(`BIN LOG`)에 기록한다.
3. `Slave` 노드의 `IO Thread`는 `Master` 노드의 로그 파일(`BIN LOG`)을 파일(Replay log)에 복사한다.
4. `Slave` 노드의 `SQL Thread`는 파일(`Replay log`)을 한 줄씩 읽으면서 데이터를 저장한다.

### 장점

- `Master node`와 `Slave node`가 각기 다른 명령을 수행하도록 함으로써 데이터베이스의 부하 분산
- `Master node`의 데이터 손상 시 `Slave node`의 데이터를 이용해 복구가 가능하므로 데이터 안정성 확보

### 단점

- 데이터가 비동기(`Asynchronous`) 방식으로 동기화되므로 데이터베이스 간 동기화가 완전히 보장되지 않아 **일관성에 문제가 발생**할 수 있음
- **마스터 노드가 정상 작동하지 않을 경우 복구 및 대처가 까다로움**

<aside>
💡

**Replication lag**

- `Master node` & `Slave node` 간 속도 차에 의한 병목 현상
    
    → Master에서 Slave로 데이터가 전달되는 찰나의 시간 차이로 인해 발생. 
    
    - `Master node` : multi-thread로 write task 수행
    - `Slave node` : single-thread로 write task 수행
- **Problem & Solution**
    - **Problem 1** : 장기 실행 쿼리
        - `Statement Based Replication` 때문에, SQL이 복제본에서 재실행되는 만큼 지연 시간이 누적된다.
        - **Solution** : `Row Base Replication` (변경된 결과를 복제) or `Mixed Based Replication` (비결정적 동작에만 RBR로 동작)을 통해 해결
            - or `tuning`, `index`을 활용
    - **Problem 2** : 쓰기 쿼리량 증가
        - 트래픽 폭증이나 배치 작업으로 인해 Master DB로 처리해야 할 쓰기 데이터가 많아질 때 발생
        - **Solution :** `Multi-Threaded Replication` 설정(Worker 스레드 증설), **샤딩** 또는 도메인 분리를 통한 데이터 경량화
    - **Problem 3** : Slave 로드 증가
        - `slave server`의 트래픽이 많아서 복제 데이터를 처리할 여력이 부족
        - **Solution :** Slave 서버를 추가로 구성 → 조회 트래픽 부하 분산
</aside>

## 3. Partitioning (데이터 분할) → 테이블을 분산!

WHEN? 서비스가 커지면서 데이터의 규모가 대용량화될때

> 테이블을 분산해서 탐색 범위를 축소시킨다!
> 

어떻게? `table`을 `partition`이라는 작은 단위로 나누어 관리한다. 

- 목적
    - 성능
        - 쿼리 성능 향상 → 대용량 `data write`에서 효율적!
        - `Full Scan`에서 `Data Access`의 범위를 줄인다.
        - in OLTP (many insert occurs) , INSERT 작업을 작은 단위인 partition들로 분산시킨다.
    - `Availability`

### Vertical Partitioning

> 열을 분리하는 파티셔닝
> 

<img width="537" height="410" alt="스크린샷 2026-02-27 오후 3 08 51" src="https://github.com/user-attachments/assets/66029ed3-fea8-4a45-82a7-35aaebf6ae43" />


- 제3정규화와 비슷 (하지만 이거는 이미 정규화된 데이터를 분리하는 것)
- 한 테이블을 select 했을때, 필요없는 컬럼이 없어지기 때문에 훨씬 많은 수의 ROW를 메모리에 올릴 수 있다.
- 하지만 데이터를 찾는 과정이 기존보다 복잡하다.

### Horizontal Partitioning

> 행을 분리하는 파티셔닝
> 

<img width="315" height="181" alt="스크린샷 2026-02-27 오후 3 09 03" src="https://github.com/user-attachments/assets/bcc1fa58-d097-4324-93a0-fc83d87bcc1c" />


- 일반적인 분산 저장 기술에서 파티셔닝 = 수평 분할!
- 데이터 개수가 작아지고 인덱스의 개수도 작아져서 성능이 향상됨.
- 하지만 서버간의 연결과정이 많아지고, 데이터를 찾는 과정이 기존보다 복잡하기 때문에 `latency`가 증가한다!

## 4. Sharding (물리적 분산) - DB 서버를 분산!

> vs `Horizontal Partitioning`?

1. `sharding`은 `db`서버를 분할하고 → ***데이터베이스*** 차원의 scale-out 가능!
2. 수평적 파티셔닝은 ***동일한 DB 내에서 테이블을 분할***한다.
> 

하나의 거대한 데이터베이스를 여러 개의 작은 단위인 샤드로 나누어 분산 저장한다.

- 목적
    - 성능 향상
        - 데이터 급증에 따른 용량 이슈와 CRUD 성능 저하 방지
    - 가용성 증대
        - 특정 DB 장애가 전체 서비스 장애로 이어지지 않도록 방지
    - 유연한 증설
        - 예측 가능한 서버 확장 방안을 마련하여 트래픽 증가에 유연하게 대응 가능

### Modular sharding

> PK를 %한 결과로 DB를 결정한다.
> 
- 장점 : **데이터가 균일하게 분산** → 리소스 활용도가 높다!
- 단점 : 하지만 DB 증설 시 기존 데이터의 Migration이 필요하다.

### Range sharding

> PK의 범위를 기준으로 DB를 결정한다.
> 
- 장점 : 증설 시 데이터 재정렬 비용이 거의 들지 않는다.
- 단점 : 특정 샤드로 데이터와 트래픽이 쏠릴 수 있다.

<aside>
💡

그럼 둘 중에 어떤 것을 선택해야 할까?

`Modular Sharding` : 데이터가 균일하게 분산되는 것이 최우선일 때! 

**(사용자 및 계좌 정보)**

→ 즉, 전체 데이터의 크기가 어느 정도 예측 가능할 때! 모든 서버에 부하가 고르게 분산되어야 하는 마스터 데이터에 적합. 

`Range Sharding` : 빠르고 유연한 확장이 우선일 떄 

**(이력 데이터)**

→ 시간에 따라 끊임없이 쌓이는 이력 데이터. (송금 내역, 카드 결제 기록)

</aside>

<img width="340" height="112" alt="스크린샷 2026-02-27 오후 3 09 22" src="https://github.com/user-attachments/assets/08ef7b46-f177-4bd9-8686-3c269b1a0f88" />


input db = write slave = read.. 등등해서 리플리케이션 -> 부하 감소
그리고 CDC (Change Data Capture) 작업 수행 → 카프카를 연동해서 데베 변동사항 감지하고 추출해서 카프카로 데이터를 실시간으로 수집하고 전달해줘서, Elastic Search 커넥터를 거쳐서 인덱싱되어서 검색해준다.

# sql injection

## 공격 유형

- **Error Based SQL Injection**
    - DB가 출력하는 **에러 메시지**를 이용해 DB 구조나 정보를 파악하는 기법
        - 일단 필드에 데이터를 넣어보고, 에러메시지를 계속 분석해서 핵심 정보 얻기
- **Union Based SQL Injection**
    - **UNION** 키워드로 원래의 쿼리에 새로운 쿼리를 합쳐서 내부 데이터를 얻어내는 기법.
        - 모든 데이터가 나오게 한다.
- **Blind SQL Injection**
    - 화면에 아무것도 안 뜰때 사용한다.
        - 로그인이 성공하면 맞고, 아니면 틀림으로 판단해서 한 글자씩 비밀번호를 알아낸다.

## 방어 기법

- **Prepared Statement 사용**
    - 사용자 입력값을 변수로 처리하여 SQL 구문을 미리 컴파일해 두는 방식
        - where id = ? → 입력된 내용을 명령어로 해석 x, 아주 이상하고 긴 이름을 가진 사용자를 찾으려고 시도한다!
- **화이트 리스트 기반 필터링**
    - `Prepared Statement`를 사용하기 어려운 경우(예: `ORDER BY` 절 등), 안전한 문자열 패턴을 미리 정의하여 허용하는 방식
        - **동작 원리**: "우리 시스템은 `최신순`, `조회순`, `이름순` 이 세 단어만 허락한다"라고 미리 정해둡니다.
        - **실제 상황**: 공격자가 정렬 조건에 몰래 SQL 명령어를 섞어 보냅니다.
        - **수비 결과**: 시스템이 명단을 확인하고 **"명단에 없는 단어네? 거절!"** 하며 입구에서 바로 막아버립니다.
- **에러 메시지 관리**
    - 서버 내부에서 오류가 나더라도 사용자는 구체적인 이유를 알려주지 않는다.
        - **나쁜 예**: "SQL 문법 오류: `member` 테이블의 `id` 컬럼 근처가 잘못되었습니다." (공격자가 테이블 이름을 알아냄)
        - **좋은 예**: "알 수 없는 오류가 발생했습니다. 잠시 후 다시 시도해주세요."

# 대용량 테이블 고려 사항

## 인덱스 설계 원칙

> 인덱스를 효율적으로 설계해야 됨!
> 
- 카디널리티
    - 중복도가 낮고 값이 다양한 열을 인덱스로 지정 → 탐색 범위가 좁혀져 성능이 극대화!
- 적정 개수 유지
    - 인덱스는 추가적인 저장 공간 차지해서, CUD 성능을 떨어트린다 → 테이블당 최대 4개 정도가 적당!
- Composite Index
    - 조건절에서 여러 열이 자주 쓰이면 → 묶어서 하나의 인덱스로!
- Covering Index
    - query에서 필요한 모든 데이터를 인덱스 자체에서 찾게한다.

## 읽기 / 쓰기 트래픽 분산

> 하나의 DB가 모든 요청을 감당하기 어려워져서 → 역할을 나누기!
> 
- `CQRS` (`Command Query Responsibility Segregation`) → 쓰기와 조회의 책임을 분리하기
    - **쓰기** 전용 DB(ex) `MySQL`)와 **읽기/검색** 전용 DB(ex) `MongoDB`, `Elasticsearch`)를 분리하여 부하를 분산
- 검색 엔진 활용
    - 특히 복잡한 검색 텍스트 처리는 역인덱싱 기술을 가진 **Elasticsearch**를 활용하는 것이 RDB의 `LIKE` 검색보다 훨씬 효율적!!

## 물리적 분산 (샤딩)

> 단일 DB 서버 한계를 넘어야 할 때
> 

위에 참고

<aside>
💡

예시 - 카드 결제 리워드 이벤트

> 특정 카드 결제 시 10% 리워드를 제공하는 이벤트로 인해 매일 수천만 건의 결제 데이터가 적재될 때
> 

1. **`CQRS`**

**쓰기** : 결제 발생 시 즉시 처리해야 하는 핵심 트랜잭션만!

**읽기** : 리워드 조회나 이벤트 랭킹 검색 → 쓰기 DB에서 발생한 데이터를 `CDC`(`Debezium` / `Kafka`)를 통해 실시간으로 복제해서 읽기 전용으로 가공한다. 

→ 리워드 내역을 조회할 때는 무거운 MySQL 대신 검색에 최적화된 `ElasticSearch`를 즉시 응답하도록!

1. **`Range Sharding`**
    
    결제 일자나 결제 번호를 기준으로 DB를 쪼갠다.
    
    ex) 1월 데이터는 1번 DB, 2월 데이터는 2월 DB
    
    → 이벤트 기간에 데이터가 폭증해도 기존 1월 데이터를 옮길 필요 없이 3월용 DB 서버만 추가하면 된다. 
    
2. **인덱스 최적화**
    
    카디널리티 활용 → 결제 승인 번호 or 사용자 id 처럼 중복이 적고 고유한 값을 인덱스로 설정
    
    ex) 승인 번호를 인덱스로 걸면 단 1건으로 범위를 좁힐 수 있다.
    
    Multi Index : [사용자ID + 결제일자]를 묶어서 인덱스로 만든다. 
    
3. **캐싱 시스템 → DB 접근 차단**
    
    Memcached/Redis → 모든 웹 서버의 메모리를 하나로 묶어서 → 거대한 캐시 저장소를 만듦.
    
    남은 리워드 총액같은 데이터는 DB까지 가지 않고, Memcached에서 바로 응답하게 해준다. 
    
</aside>

# Statement & PreparedStatement

## Statement

> JDBC에서 제공하는 가장 기본적인 쿼리 실행 방식
> 

**작동 방식 :** SQL 문에 변수 값을 직접 포함하여 데이터베이스에 전달

*쿼리 전송 → 문법 검사(파싱) → 실행 계획 생성 → 결과 반환의 4단계를 매번 수행한다. *****

캐시 사용 x

실행할 때마다 데이터베이스가 SQL을 파싱 & 컴파일 → 반복 실행 시에 효율이 떨어짐. 

## PreparedStatement

> Statement의 성능 한계를 극복하기 위해 개발된 향상된 인터페이스
> 

**작동 방식 :** 플레이스홀더(`?`)를 사용하여 쿼리 틀을 먼저 만들고, 나중에 실제 값을 바인딩

- **실행 과정**:
    - **최초 실행**: SQL 전송 및 파싱, 컴파일, 실행 계획 생성을 수행하여 캐시에 저장
    - **재사용 시**: 준비된 쿼리에 값만 바인딩하여 즉시 실행
- **특징**: 동일한 쿼리를 반복 실행할 때 매우 효율적이며, **SQL 인젝션 공격을 예방**하는 데 필수적이다.
