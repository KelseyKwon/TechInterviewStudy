> 트랜잭션: 여러 데이터베이스 연산을 하나의 논리적 작업 단위로 묶어서, 전부 성공하거나 전부 실패하게 만드는 매커니즘
> 

## 트랜잭션 정의와 필요성

- 하나 이상의 SQL 연산(INSERT/UPDATE/DELETE 등)을 묶어서 처리하는 논리적 작업 단위
- 중간에 장애가 나더라도 DB가 일관된 상태를 유지하도록 보장하는 게 핵심
- 금융·의료 같은 정합성이 중요한 시스템에서 필수적이고, MySQL, PostgreSQL, Oracle 등 주요 RDBMS가 ACID 트랜잭션 지원
- 예시) A가 B에게 5만원 송금할 때
    1. A 계좌에서 5만원 차감
    2. B 계좌에서 5만원 증가
    
    ⇒ 두 연산이 둘다 성공하거나 둘 다 취소되어야 함
    
    ⇒ 이 전체가 하나의 트랜잭션
    

## Autocommit

- 대부분 DB(MySQL, PostgreSQL, Oracle)의 기본값 = ON
- 각 SQL 한 문장이 자동으로 하나의 트랜잭션이 됨 (BEGIN + 실행 + COMMIT 자동)
- 예) `UPDATE account SET balance = balance - 50000 WHERE id = 1;`
- 예시)
    - Autocommit = ON 상태에서 여러 SQL을 순서대로 실행하면, 중간에 장애가 나도 이전 SQL들은 이미 커밋돼서 롤백 불가
    - 해결: `SET AUTOCOMMIT = 0; BEGIN; … COMMIT/ROLLBACK;` 또는 Spring @Transactional 같은 프레임워크 사용
    - 실무에서 “트랜잭션이 안 묶인다” 문제의 90%가 Autocommit 때문
        
        ⇒ `@Transactional`은 내부적으로 Autocommit을 끄고 명시적 트랜잭션 관리
        

## Commit이 실제로 하는 일

- 단순 확정이 아니라
    1. 변경 내용을 Redo Log에 기록 (변경 재현 가능하게)
    2. 로그를 디스크에 flush (fsync) - 여기서 Durability 보장
    3. 트랜잭션 완료 표시 후 자원 해제
- 예시)
    - 송금 커밋 시 Redo Log에 “A-5만, B+5만” 기록 후 fsync
    - 서버 다운 후 재시작해도 로그 따라 복구 → 잔액 변화 영속화

## Rollback이 실제로 하는 일

- Undo Log에 저장된 이전 값을 기반으로 상태 복원
- InnoDB는 UPDATE 전에 이전 행 데이터를 Undo Log에 자동 저장 → 롤백 시 역순 적용

```
Active ──(성공)──→ Partially Committed ──(Redo Log fsync)──→ Committed
  ↓
(Failed)
  ↓
Aborted ──(Undo Log 적용)──→ Rollback 완료
```

## 트랜잭션이 가져야 할 ACID

### Atomicity (원자성)

- 트랜잭션에 포함된 연산은 모두 수행되거나, 전혀 수행되지 않음(all-or-nothing) 의미
- 실패 시 `ROLLBACK`으로 트랜잭션 시작 이전 상태로 되돌림
- 예시) 송금 중 B 계좌에서 `UPDATE`에서 에러가 나면, A 계좌에서 빠져나간 5만원도 자동으로 롤백되어야 함
    
    ⇒ 일부만 반영되면 돈이 사라지는 상황 발생
    

### Consistency (일관성)

- 트랜잭션 전후로 데이터는 항상 유효한 상태(제약 조건을 모두 만족)여야 함
- 기본키, 외래키, 체크 제약, 비즈니스 규칙 등을 깨지 않아야 함
- 예시) 전체 은행 시스템에서 모든 고객 예금 총합이 비즈니스 규칙상 맞아야 한다고 하면,
송금 전후로 총합은 변하지 않아야 함 (수수료 등 정책 제외)
- Consistency의 진짜 의미
    - Atomicity, Isolation, Durability: DB가 기술적으로 보장
    - Consistency: DB가 자동 보장 안함. 애플리케이션 + 제약 조건이 담당
        - `CHECK (balance ≥ 0)` 없으면 Atomicity만으로도 음수 잔액 가능
- 예시)
    - 송금 트랜잭션은 Atomic하게 실행되지만, “”잔액 < 이체 금액이면 안된다” 같은 비즈니스 규칙은 애플리케이션 로직에서 체크해야 함
    - DB는 제약 조건 위반만 막아줄뿐!!!

### Isoaltion (고립성)

- 동시에 여러 트랜잭션이 실행되어도, 각 트랜잭션은 마치 자기 혼자 실행되는 것처럼 느껴져야 함
- 중간(커밋 안된) 결과를 다른 트랜잭션이 보지 못하게 해서 이상 현상(Diry Read, Non-repeatable Read 등)을 방지
- 예시) 두 사람이 동시에 같은 계좌에서 돈을 뽑을 때, 서로가 중간 잔액을 잘못 보고 과도 인출하는 것을 막아야 함

### Durability (지속성)

- 한번 커밋된 트랜잭션의 결과는 시스템 장애(전원 off 등) 후에도 유지되어야 한다는 성질
- 이를 위해 Write-Ahead Logging(WAL), Undo/Redo 같은 로그 기반 복구 기법 사용
- 예시) 송금 완료 후 갑자기 서버가 다운되더라도, 재부팅 후에도 송금 결과(잔액, 거래내역)가 그대로 남아 있어야 함

## Isolation Level과 이상 현상

- 동시성과 성능 때문에 Isolation을 레벨로 나눠서 조절
- 대표적 이상현상
    - Dirty Read
        - 다른 트랜잭션이 아직 커밋하지 않은 변경을 읽는 상황
        - 다른 트랜잭션이 A 계좌 잔액을 100만 → 0으로 만들었다가 롤백했는데, 그 0을 기준으로 출금 가능 여부를 판단
    - Non-repeatable Read
        - 같은 쿼리를 2번 했는데, 그 사이 다른 트랜잭션 커밋으로 결과가 달라지는 상황
        - 잔액 조회를 2번 했는데 중간에 다른 트랜잭션이 입금/출금을 커밋해서 내 트랜잭션 내에서 잔액이 튐
    - Phantom Read
        - 조건에 맞는 행의 집합이 다른 트랜잭션 삽입/삭제로 늘거나 줄어드는 상황
        - 잔액 0원인 계좌 리스트를 조회하고 정산 처리 중인데, 다른 트랜잭션이 중간에 새로운 0원 계좌를 만들어서 2번째 조회 때 목록이 달라지는 상황
- ANSI SQL Isolation Level
    
    
    | Isolation Level | Dirty Read | Non-repeatable Read | Phantom Read |
    | --- | --- | --- | --- |
    | Read Uncommitted | O | O | O |
    | Read Committed | X | O | O |
    | Repeatable Read | X | X | O |
    | Serializable | X | X | X |
    - Read Uncommitted: 가장 낮은 격리, Dirty Read 허용. 실무에서는 거의 사용하지 않음
    - Read Committed: 커밋된 데이터만 읽음, 대부분 상용 DB의 기본값 (예: Oracle)
        - 매 SELECT마다 최신 커밋 버전 읽음 → Non-repeatalbe Read 발생 가능
    - Repeatable Read: 같은 행을 여러번 읽어도 값이 변하지 않음
        - 트랜잭션 시작 시점 스냅샷 고정 → Non-repeatable Read 방지
        - 예시) 잔액 조회 중 다른 트랜잭션이 입금하면,
            
            Read Committed는 두 번째 조회에서 새 잔액 보지만 Repeatable Read는 시작 시점 잔액 유지
            
    - Serializable: 완전 직렬 실행과 같은 효과, 하지만 동시성이 떨어지고 재시도가 필요할 수 있음
    - 예시) 송금·입출금 같은 핵심 거래는 Non-repeatable Read나 Dirty Read가 발생하면 안 되기 때문에 최소 Read Committed 이상, 중요한 정산·마감 배치 등은 Serializable과 비슷한 강한 격리를 사용
    - Phantom Read
        - MySQL InnoDB: Repeatable Read에서 Next-Key Lock으로 많은 Phantom 막음
        - PostgreSQL: MVCC 스냅샷 기반 → Serializable은 SSI(Serializable Snapshot Isolation)로 보장
        - Phantom/보장 수준은 DB 구현(MVCC/락 전략)에 따라 다르게
- 기타 이상현상
    - Lost Update: 두 트랜잭션의 업데이트가 서로 덮어쓰는 문제
    - Write Skew: 두 트랜잭션이 서로 다른 행을 수정하지만, 결과적으로 제약조건 위반
        - 의사 2명이 동시에 환자 A/B에게 약 처방 → 총 약물 상한 초과

## 은행 송금 예제로 보는 트랜잭션 흐름

1. 트랜잭션 시작
    
    `BEGIN TRANSACTION;`
    
2. 출금 계좌 잔액 확인
    
    `SELECT balance FROM account WHERE id =:from;`
    
    잔액 < 이체금액이면 `ROLLBACK;` 후 “잔액 부족” 에러
    
3. 출금 처리
    
    `UPDATE account SET balance = balance - :amount WHERE id =:from;`
    
4. 입금 처리
    
    `UPDATE account SET balance = balance + :amount WHERE id =:to;`
    
5. 거래 내역 기록
    
    `INSERT INTO transaction_history …;`
    
6. 커밋
    
    모든 단계 성공 시 `COMMIT;` → ACID 보장
    
- 중간에 네트워크 장애나 DB 오류가 나면 DBMS가 트랜잭션을 롤백해서 둘 중 하나만 반영된 상태가 남지 않도록 보장
- 실제 은행에서는 단일 DB안에만 해당 로직이 있는 것이 아니라
    - 코어뱅킹 DB
    - 외부 지급결제망(ACH, RTGS, 카드사 등)
    - 로그/이벤트 시스템
    
    여러 시스템과 연관돼서, 분산 트랜잭션 / 사가 패턴과 같은 설계도 같이 고려됨
    

## 분산 트랜잭션 - 2PC + Saga

### 2PC (Two-Phase Commit)

- 분산 시스템에서 원자성 보장 프로토콜
1. Prepare Phase: 코디네이터가 각 노드에 커밋 가능 여부를 물음 → 자원 예약 후 Yes/No 응답
2. Commit Phase: 모두 Yes면 Commit 지시, 아니면 모두 Abort
- 예시
    - A은행(노드1) → B은행(노드2) 100만원 이체
        - Prepare: A 출금 예약, B 입금 예약
        - 모두 OK면 동시 커밋 → 부분 업데이트 불가
- 단점: 코디네이터 단일 실패점, 지연 발생

### Saga 패턴 (마이크로서비스 대안)

- 각 서비스가 로컬 트랜잭션만 하고, 실패 시 보상 트랜잭션(Compensation) 실행
- 예시
    1. 출금 서비스: A 계좌 -100만 (성공)
    2. 입금 서비스: B 계좌 +100만 (실패)
    3. 보상: 출금 취소 (+100만 복원)
- 장점: 2PC보다 느슨하지만, 장애 복구 쉬움. Idempotent(재실행 안전) 설계 필수

### 선택기준

- 2PC: 강한 원자성 필요 + 참여자 소수 + 지연/블로킹 감수 가능
- Saga: 고가용성/확장성 우선 + 실패를 보상으로 처리 + Eventual Consistency 수용
- 예시
    - 내부 코어뱅킹: 2PC (강한 일관성)
    - 마이크로서비스 + 외부 은행 연동: Saga (장애 복구 쉬움)

---

## Lost Update

- 두 개의 트랜잭션이 동시에 같은 데이터를 수정할 때, 먼저 수정된 내용이 나중에 수정된 내용에 의해 덮어씌워져 사라지는 현상
- 예시) 계좌 잔액 100원
    1. 트랜잭션 T1: 잔액 100 읽음
    2. 트랜잭션 T2: 잔액 100 읽음
    3. T1: 50원 입금 → 150으로 UPDATE (커밋)
    4. T2: 30원 입금 → (자기 기준 100 + 30) = 130으로 UPDATE (커밋)
    5. 결과: 최종 잔액은 130원이 됨 (A의 50원 분실)
        1. 원래 기대: 100 + 50 + 30 = 180이어야 하는데, 실제 최종 잔액은 130 → T1의 50원이 Lost
- 해결책
    - Pessimistic Lock (비관적 락)
        - 어차피 충돌 날 가능성이 높다 가정 → 읽을 때부터 락을 걸어버리는 방식
        - 데이터를 읽을 때부터 락을 걸어 다른 사람이 못 건드리도록
        - 읽기 시점에 `SELECT … FOR UPDATE` 등으로 X-Lock을 잡으면, 다른 트랜잭션은 해당 행을 수정/읽기 못하게 막음
    - Optimistic Lock (낙관적 락)
        - 충돌은 가끔 일어난다고 가정 → 일단 다 같이 읽고 커밋 직전에 충돌 여부 체크
        - 수정 시점에 내가 읽었던 버전이 맞는지 확인 (Version Column 사용)
        
        ```sql
        UPDATE account 
        SET balance = balance + 30000, 
            version = version + 1 
        WHERE id = 1 AND version = 5;  -- 내가 읽었던 버전 확인
        ```
        
        ⇒ 영향 행 수가 0이면 충돌 → 재시도 or 오류
        

## Locking

- 데이터의 일관성을 위해 건들지 않도록 표시하는 기법

| 요청 락 \ 기존 락 | S-Lock | X-Lock |
| --- | --- | --- |
| S-Lock | O | X |
| X-Lock | X | X |
- Shared Lock (S-Lock, 공유 락)
    - 다른 트랜잭션도 읽기는 가능하지만, 쓰기는 불가능
    - 여러개 가능
- Exclusive Lock (X-Lock, 배타 락)
    - 다른 트랜잭션은 읽기도 쓰기도 모두 불가능
    - 단독 독점
- 스토리지 엔진 락 (MySQL InnoDB 기준)
    - 어느 범위까지 락을 걸 것인가
    - Row-level Lock
        - 특정 행(Row)만 잠금
        - 동시성이 높지만 관리가 복잡함
        - InnoDB DML 기본
    - Table-level Lock
        - 테이블 전체를 잠금 (DDL, MyISAM 등)
        - 관리 비용은 낮지만 동시성이 최악 (데이터 변경 작업 DDL시 발생)
    - Gap Lock
        - “레코드와 레코드 사이의 빈 공간(갭)”에 락.
        - 특정 범위 안에 새로운 행이 끼어드는 것을 막아 Phantom Read를 줄임.
    - Next-Key Lock
        - “레코드 락 + 앞쪽 갭 락” 묶음.
        - `SELECT ... WHERE amount BETWEEN 100 AND 200 FOR UPDATE;` 시, 해당 범위의 기존 행과 그 사이 갭까지 잠가서, 중간에 새로운 행이 끼어들지 못하게 함 → Phantom Read 방지.
    - InnoDB vs MyISAM ⇒ 왜 InnoDB?
        - MyISAM 트랜잭션 불가 이유: **Undo/Redo 기반 트랜잭션 로그가 없고**, Crash recovery/rollback을 지원하지 않아 트랜잭션을 제공하지 않음→ 사용하기 어려움..
        
        | 기능 | InnoDB | MyISAM |
        | --- | --- | --- |
        | 트랜잭션 지원 | O | X (Undo/Redo 로그 없음, 롤백 불가) |
        | Row Lock | O | X (Table Lock) |
        | 외래키(FK) | O | X |
        | Crash Recovery | O | X |
- Deadlock
    - 서로 자원을 기다리다 서로 영원히 멈추는 상황
        
        ```sql
        T1: UPDATE accountA; UPDATE accountB;
        T2: UPDATE accountB; UPDATE accountA;
        ```
        
        → T1이 A 락 잡고 B대기, T2가 B 락 잡고 A 대기 ⇒ 데드락
        
    - 해결
        - 순서 다르게 락 획득 시 발생 → InnoDB가 자동 감지 (wait-for graph 사용) 후, 작은 트랜잭션(변경 행 적은 쪽)을 강제 Rollback
        - 예방: 일관된 자원 획득 순서 (항상 A → B 순서로 락)

## MVCC (Multi-Version Concurrency Control)

> **MVCC는 읽기 동시성을 높이는 기술이고, Lock은 쓰기 충돌 제어 기술. 둘은 대체가 아니라 보완 관계!!**
> 
- 현대적인 DB(MySQL, PostgreSQL 등)가 읽기 작업은 쓰기 작업을 기다리지 않는다를 구현하는 핵심 기술
- 핵심 원리
    - 데이터를 수정할 때 기존 데이터를 덮어쓰는게 아니라, 이전 버전을 별도의 공간(Undo Log 등)에 보관
    - SELECT 시점에 내 트랜잭션이 볼 수 있어야 하는 버전을 골라서 읽게 함
        
        ⇒ 스냅샷 읽기
        
- 작동 방식
    1. 어떤 행을 UPDATE 하기 전에, 현재 버전을 Undo Log에 기록.
    2. 트랜잭션 T1이 SELECT를 할 때
        - 현재 레코드가 T1이 시작된 이후에 수정된 것이라면, Undo Log에 있는 **옛 버전**을 읽음.
    3. 그래서 누군가 UPDATE 중이더라도, 다른 트랜잭션은 “자기 시작 시점 기준의 스냅샷”을 읽기 때문에 락 대기가 필요 없음.
- 장점
    - SELECT가 UPDATE/DELETE를 거의 막지 않고, 반대로도 마찬가지라 **동시성·성능이 크게 향상**.
    - Repeatable Read 같은 격리 수준을 구현하기 쉬워짐
    - 거래 내역 조회는 MVCC 덕분에 실시간 입출금 UPDATE와 섞여도 거의 락 경합 없이 안정적으로 실행 가능
    - 다만 실시간 현재 잔액 같은 강한 일관성이 필요한 조회는 적절한 락/격리수준을 함께 사용해야 함
- MVCC 동작별 락 여부
    
    
    | 작업 | MVCC 동작 | 락 필요 여부 |
    | --- | --- | --- |
    | SELECT | Undo Log 기반 스냅샷 읽기 | X (락리스) |
    | UPDATE | 기존 행에 X-Lock | O |
    | INSERT | Gap/Next-Key Lock 가능 | O |
    - MVCC ≠ No Lock: 읽기는 락리스지만, 쓰기는 여전히 X-Lock 필요

---

## Join

- 여러 테이블을 하나의 결과 집합으로 묶는 연산

### Logical Join

- 어떤 결과를 반환할지

| **종류** | **설명** | 예시 |
| --- | --- | --- |
| **Inner Join** | 양쪽 테이블에 모두 매칭되는 행만 반환 | account와 customer 조인
→ 실제 존재하는 계좌 + 고객 정보만 조회 |
| **Left (Outer) Join** | 왼쪽 테이블의 모든 행 + 오른쪽의 매칭되는 행 (없으면 NULL) | 고객 목록을 기준으로 계좌가 없는 고객도 함께 보고 싶을 때 |
| **Full Outer Join** | 양쪽 모두의 합집합 (MySQL은 지원 안 해서 UNION 사용 필요) | 계좌 테이블과 해지된 계좌 이력 테이블을 모두 포함해서 보고 싶을 때 |
| **Cross Join** | 카테시안 곱 (모든 경우의 수 결합) |  |

### Physical Join

- 실제 실행 방법
- Nested Loop Join
    - 2중 for문과 같은 방식
    - 인덱스가 잘 잡힌 작은 테이블을 바깥쪽(OUTER)으로 두면 효율적.
- Hash Join
    - 한쪽 테이블을 해시 테이블로 만든 뒤, 다른 쪽을 스캔하며 해시로 매칭.
    - 대용량 조인에 유리, PostgreSQL 등에서 흔히 사용.
- Sort Merge Join
    - 양쪽 데이터를 정렬한 뒤 병합
    - 인덱스가 없거나 범위 조인에 쓸 만한 차선책

### 조인 순서 최적화

- 쿼리: `A JOIN B JOIN C` → 옵티마이저가 cost 최소화 순서로 재배열
- 기준: 테이블 크기, 인덱스, 통계 정보

### 인덱스와의 관계

- Nested Loop Join 최강 케이스: Outer 테이블 작고, Inner에 적절한 인덱스 있을 때
- 예시) 고객(작은 테이블) 기준으로 계좌(인덱스 있는) 조인
