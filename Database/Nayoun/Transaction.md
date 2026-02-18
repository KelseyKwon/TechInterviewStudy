# 트랜잭션이란?

> 한꺼번에 수행되어야 할 일련의 연산 모음
→ 하나의 작업을 위해 더 이상 분할될 수 없는 명령들의 모음!
>


<aside>
📎 예시로, **계좌 이체**가 있다.

***계좌 A에서 계좌 B로 50달러를 보내는 상황이라고 가정***해보자. 

그러면, `계좌 A에서 돈이 50달러 인출되는 상황과 계좌 B로 돈이 50달러 입금되는 상황` 2가지가 있다. ⇒ **인출 & 입금**

근데, 만약에 인출에는 성공했는데, 입금에는 실패하면 상황이 맞지 않는다.

⇒ 따라서, 두 개의 연산이 동시에 성공하던지, 동시에 실패하던지가 되어야 한다

⇒ 이렇게 동시에 묶는 방법이 바로 트랜잭션!

</aside>



```java
START TRANSACTION
    -- 이 블록안의 명령어들은 마치 하나의 명령어 처럼 처리됨
    -- 성공하던지, 다 실패하던지 둘중 하나가 됨.
    A의 계좌로부터 인출;
    B의 계좌로 입금;
COMMIT
```

> 따라서 DB는 데이터를 읽어온 후 다른 테이블에 데이터를 입력하거나, 갱신 or 삭제하는 도중에 오류가 발생하면, 결과를 재반영하는 것이 아니라, 모든 작업을 원상태로 복구 & 처리 과정이 모두 성공해을 때만 결과 반영!
> 

## MYSQL Transaction

> In MySQL, `Transaction`은 데이터베이스의 상태를 바꾸는 일종의 작업단위이다!
> 

`INSERT`, `DELETE`, `UPDATE` 등의 SQL 명령문을 실행

→ 내부적으로 자동적으로 Commit 실행 

→ 변경된 내역을 DB에 반영.

즉, 

```sql
INSERT INTO user VALUES (...);
```

을 실행할때마다 → 내부적으로 자동으로 Commit이 실행된다!

<aside>
⚠️ 하지만 꼭 질의어 한 줄이 트랜잭션 하나의 단위는 아니다!

</aside>

```sql
/**
1. A 계좌에서 10만원 차감
2. B 계좌에 10만원 추가
*/
START TRANSACTION;

UPDATE account SET balance = balance - 100000 WHERE id = 'A'; -- 자동 COMMIT
UPDATE account SET balance = balance + 100000 WHERE id = 'B'; -- 에러 발생

COMMIT;
```

처럼 `AUTO COMMIT`을 끄고, 사용자가 직접 트랜잭션 단위를 지정해줘야 하는 경우도 있다. 

## ACID

- **`Atomicity`**(원자성)
    - 트랜잭션의 모든 작업은 “전부 반영”되거나 “아예 안 반영”되어야 한다.
- **`Consistency`**(일관성)
    
    <img width="520" height="187" alt="스크린샷 2026-02-10 오후 5 42 27" src="https://github.com/user-attachments/assets/f4dabcf4-95c7-494f-b2c1-3b1df2dacd6f" />

    
    - 트랜잭션 실행 전/후에 데이터베이스가 항상 일관성 유지
        - 위의 예에서는 A와 B의 총합이 transaction 실행 전후에 동일하게 유지되어야 함을 의미
- **`Isolation`**(고립성)
    - 여러 트랜잭션이 동시에 실행돼도 각 트랜잭션은 다른 트랜잭션의 중간 결과를 절대 볼 수 없어야 한다.
    - 마치 순차적으로 실행한 것처럼 보여야 함.
- **`Durability`**(영속성)
    - 트랜잭션이 성공적으로 끝나면 시스템이 꺼져도 변경 사항이 절대 사라지지 않아야 한다
        - 계좌 이체 예시에서, 사용자에게 transaction이 완료되었다고 통지한 후에는($50 이체가 이루어졌음), 소프트웨어나 하드웨어 failure가 발생하더라도 데이터베이스 업데이트가 지속되어야 한다.

## Commit & Rollback

<aside>
📖

### Commit

- 모든 작업들을 처리하겠다고 확정하는 명령어.
    - 해당 처리 과정을 DB에 영구 저장하겠다!
    - Commit 수행 → 이전 데이터가 완전히 반영됨 → UPDATE!
</aside>

<aside>
📖

### Roll-back

- 작업 중 문제가 발생, 트랜잭션 처리 과정에서 발생한 변경 사항을 취소하는 명령어!
    - 시작되기 이전의 상태로 되돌아감.
</aside>

## 트랜잭션 상태

<img width="526" height="377" alt="스크린샷 2026-02-10 오후 5 42 40" src="https://github.com/user-attachments/assets/5061496e-1c34-46c6-8814-f82ba1576404" />


- **`Active`**(활동 중)
    - 초기 상태, 트랜잭션이 실행 중인 상태 (아직 끝나지 않음)
- **`Partially Committed`**(부분 커밋)
    - 마지막 명령(`statement`)까지 실행했지만
        - 아직 완전히 커밋(확정 저장)은 안 된 상태
    - 즉, 설계된 작업대로 다 되었다고 해서, 무조건 반영하는 것이 아니라 설계자의 최종 승인이 있을 때까지 실제 DB에 작업 내용을 반영하지 않고 기다린다!
- **`Failed`**(실패)
    - 실행 도중 에러나 시스템 장애 등으로 정상 실행이 불가능하다고 판정된 상태
- **`Aborted`**(중단됨)
    - 트랜잭션이 **실패**해서 **롤백(원래 상태로 복구)**이 끝난 상태
    - 이후 두 가지 선택 가능:
        1. 재시도(`restart`): 내부 논리적 오류가 없다면 다시 시도 가능
        2. 완전히 종료(`kill`): 더 이상 시도하지 않고 완전히 중단
- **`Committed`**(커밋됨)
    - 트랜잭션이 완전히 **성공**해서
        - 모든 변경 내용이 확정적으로 DB에 반영된 상태

## 격리 수준

> 동시에 실행되는 트랜잭션을 얼마나 서로 못 보게 할 것인가? 를 정하는 기준
> 

| Isolation Level | `Dirty Read`
(= 커밋 안 된 데이터 읽음) | `Nonrepeatable Read`
(= 같은 데이터 읽었는데 값이 바뀜) | `Phantom Read`
(= 조건 조회 결과의 행 개수가 바뀜) | `Serialization Anomaly` |
| --- | --- | --- | --- | --- |
| Read uncommitted | O | O | O | O |
| Read committed | X | O | O | O |
| Repeatable read | X | X | O | O |
| Serializable | X | X | X | X |
- **`Read uncommitted`**: 커밋 안 된 데이터도 읽을 수 있음 (가장 약한 격리, dirty read 허용)
    
    ```sql
    T1: UPDATE balance = 0 (아직 COMMIT 안 함)
    T2: SELECT balance → 0 (더티 리드)
    T1: ROLLBACK → 실제 balance는 원래 값
    ```
    
- **`Read committed`**: 커밋된 데이터만 읽음, 하지만 반복 조회 시 값이 달라질 수 있음
    
    ```sql
    T1: SELECT balance → 100
    T2: UPDATE balance = 200; COMMIT;
    T1: SELECT balance → 200
    ```
    
- **`Repeatable read`**: 같은 레코드 반복 조회 시 항상 같은 값, **단 `insert`에는 완전하지 않음** (phantom read 발생 가능)

```sql
T1: SELECT * FROM order WHERE price > 100; → 3 rows
T2: INSERT INTO order (price=150); COMMIT;
T1: SELECT * FROM order WHERE price > 100; → 4 rows
```

- **`Serializable`**: 완전한 격리. 팬텀 현상 포함 모든 이상현상 방지. (가장 강함, but 성능은 가장 떨어짐)
    - 여러 트랜잭션이 **동시에 실행**되더라도, 결과가 반드시 어떤 순서의 **직렬 실행**과 동일하다!
        - 이렇게 되면 데이터베이스 상태는 항상 일관성을 유지하게 된다.

## Lost Update

> 두 개 이상의 트랜잭션이 같은 데이터를 읽고 수정할 때, 나중에 커밋된 변경이 이전 변경을 덮어써버리는 문제!
> 

**for example,**

```sql
balance = 1

// 트랜잭션 A
읽음: balance = 1
+1 지급 → 2로 만들 예정

// 트랜잭션 B
읽음: balance = 1
-1 사용 → 0으로 만들 예정

/**
1. SELECT balance
2. balance ± 1 (애플리케이션에서 계산)
3. UPDATE balance = 계산된 값
*/
```

- 위에 상황에서는, A의 갱신이 사라진다 → 갱신이 분실됨!
- 그럼 이것을 Serializable로 안될까?
    - 안됨! → Deadlock이 발생함…
        - **`SELECT`에도 락이 걸림**
        - `SELECT = SELECT FOR SHARE` 효과
        - `UPDATE` 시 서로 락 대기 발생
    
    | **Tx1** | **Tx2** |
    | --- | --- |
    | SELECT → 공유락 | SELECT → 공유락 |
    | UPDATE → 배타락 요청 | UPDATE → 배타락 요청 |
    | 서로 대기 | 서로 대기 |

→ 진정한 솔루션은 밑에

```sql
UPDATE member
SET balance = balance + 1
WHERE id = 1;
```

- **읽지 말고, DB에게 바로 계산하게 하자!**

<aside>
📖 즉, 갱신 분실은 트랜잭션 범위를 넘어서는 문제이다. (두 개의 트랜잭션이 정상적으로 커밋되었는데도 발생했기 때문에)

1. 마지막 Commit만 인정
2. 최초의 Commit만 인정
3. 충돌하는 갱신 내용 병합
</aside>

## Locking

> 동시성 제어의 핵심 요소, 
여러 트랜잭션이 동시에 데이터에 접근할 때 발생할 수 있는 **충돌 방지** & **데이터의 일관성과 무결성**을 유지!
> 

<aside>
📖 - `Exclusive(X) Lock`:
    - 데이터 읽기/쓰기 둘 다 가능 (수정할 때 필요)
    - 같은 데이터에 X락이 걸려 있으면 다른 트랜잭션은 접근 불가
- `Shared(S) Lock`:
    - 데이터 읽기만 가능 (조회만 할 때)
    - 여러 트랜잭션이 동시에 S락을 가질 수 있음(서로 읽기만 가능)
</aside>

### Lock 종류

[동시성 문제 해결하기 V1 - 낙관적 락(Optimistic Lock) feat.데드락 첫 만남](https://velog.io/@znftm97/%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-V1-%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BDOptimisitc-Lock-feat.%EB%8D%B0%EB%93%9C%EB%9D%BD-%EC%B2%AB-%EB%A7%8C%EB%82%A8)

1. **Optimistic Lock**
    
    > DB의 Lock을 사용하지 않고, Version 관리를 통해 Application 레벨에서 처리.
    > 
    - 장점
        - 충돌이 안난다는 가정하에, 별도의 락을 잡지 않으므로 Pessimistic Lock 보다는 성능적 이점을 가진다.
    - 단점
        - 업데이트가 실패했을 떄, 재시도 로직을 개발자가 직접 작성해 주어야 한다.
        - **충돌이 빈번하게 일어나거나 예상이되면, 롤백처리를 해주어야하기 때문에 Pessimistic Lock 이 더 성능이 좋을 수**도 있다.
        
        <aside>
        📖 in JPA, `@Version`을 이용해서 버전 관리 기능을 제공해준다. 
        
        - `@Version`추가 → 엔티티를 수정할 때마다 버전이 하나씩 자동으로 증가됨.
            - 이렇게 함으로써, 최초 커밋만 인정되는 방식을 구현할 수 있다!
        
        <img width="433" height="248" alt="스크린샷 2026-02-10 오후 5 43 23" src="https://github.com/user-attachments/assets/f12319c7-48a0-4372-9ba4-19f23dc15ad0" />

        
        </aside>
        
2. **Pessimistic Lock**
    
    > 자원 요청에 따른 동시성 문제가 있을 것이라고 예상하고 락을 걸어버리는 방식
    > 
    - 장점
        - 충돌이 빈번하게 일어난다면 롤백의 횟수를 줄일 수 있기 때문에, Optimistic Lock 보다는 성능이 좋을 수 있다
        - 비관적 락을 통해 데이터를 제어하기 때문에 데이터 정합성을 어느정도 보장할 수 있다.
    - 단점
        - 데이터 자체에 별도의 락을 잡기때문에 동시성이 떨어져 성능저하가 발생할 수 있다.
        - 특히 **읽기가 많이 이루어지는 데이터베이스의 경우**에는 손해가 더 크다.
        - 서로 자원이 필요한 경우, 락이 걸려있으므로 **데드락**이 일어날 가능성이 있다.

[[Spring] Redis(Redisson) 분산락을 활용하여 동시성 문제 해결하기](https://velog.io/@juhyeon1114/Spring-RedisRedisson-%EB%B6%84%EC%82%B0%EB%9D%BD%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%98%EC%97%AC-%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0)

> 하지만 Pessimistic Lock, Optimistic Lock 모두 한계점들이 명확하기 때문에, Redis와 같은 것을 활용해서 분산 락을 구현한다!
> 

## Storage Engine Lock

> MySQL에서는 스토리지 엔진 레벨과, MySQL 엔진 레벨로 나눌 수 있다
> 

### 1. Record Lock

> 일반적으로 레코드 락은 **테이블 레코드 자체를 잠그는 락**이나 MySQL에서의 레코드 락은 **인덱스의 레코드를 잠근다.**
> 
- 왜 인덱스의 레코드를 잠금?
    - 조건에 맞는 레코드를 잠글때, 그 레코드를 찾기 위해 사용하는 것이 인덱스!

```sql
-- first_name 단일 인덱스만 존재
UPDATE employees
SET hire_date = NOW()
WHERE first_name = 'Georgi'
  AND last_name = 'Klassen';
```

1. first_name = ‘Georgi’로 , 인덱스를 사용해서 빨리 찾을 수 있다. 
    1. 256개의 레코드가 나옴!
2. last_name에는 인덱스가 없으므로, 전부 하나씩 읽어보면서 비교해야 한다. 
    1. first_name = 'Georgi' 인덱스에 걸린 **256개 인덱스 레코드 전부에 락**
    2. 그중에서
        1. last_name = 'Klassen'인 **1개만 실제로 UPDATE**
        2. 나머지 255개는 “검사만 하고 버림”
    <img width="519" height="337" alt="스크린샷 2026-02-10 오후 5 43 37" src="https://github.com/user-attachments/assets/25048653-5f0a-419c-97a5-bfdbf2762455" />

    
    
3. 만약에 first_name에도, last_name에도 인덱스가 없다면 → full-table-scan이 일어난다!

### 2. Gap Lock

> 레코드가 아닌 **레코드와 레코드 사이의 간격을 잠금**으로써 레코드의 생성, 수정 및 삭제를 제어한다.
> 

<img width="448" height="490" alt="스크린샷 2026-02-10 오후 5 43 49" src="https://github.com/user-attachments/assets/cfcc7059-de6d-41bd-8517-92a581e9a092" />


- ex) J로 시작하는 레코드가 Jo, Joe 2개가 있는데, 언제든지 다른 데이터들 (Jang, Jeong, Jung)이 추가될 수 있다.
    - 따라서, 다른 트랜잭션에서 임의의 데이터가 추가되지 않도록 잠그기 위해, Gap Lock을 사용한다!
- phantom read을 방지할 수 있다.

### 3. Next Key Lock

> ‘현재 존재하는 행 + 그 주변의 빈 공간까지’ 같이 잠가서 동시 실행 중에 결과가 달라지는 걸 막는다.
> 
- **Prob :** Replication 서버에서는, select 결과가 달라질 수 있는 상황이 나온다.
    - Source Server & Replica Server 실행 순서가 달라지면, 결과가 서로 달라질 수 있다.
- **Sol :** 그래서, 그 범위에 새로운 row가 들어오지 못하도록 막는다!

하지만 next key lock도 너무 많이 잠가버려서, 효율이 안좋을 수 있다…

그래서, binary log format이 나옴!

1. `STATEMENT` 기반 Binary Log
    - “이 SQL을 실행했다”만 기록
    - 리플리카에서도 **똑같은 SQL을 다시 실행**
2. `ROW` 기반 Binary Log
    - “이 row가 이렇게 바뀌었다”를 기록
    - 리플리카는 SQL을 다시 실행 안 함
    - 그냥 **결과만 적용**

### 4. Auto-Increment Lock

> AUTO_INCREMENT 값을 할당하는 순간에만 테이블 수준으로 아주 짧게 거는 잠금!
> 

- 실무에서는 락이 어떻게 쓰일까?
    
    
    | **상황** | **왜 락이 필요함?** | **주로 쓰이는 락** |
    | --- | --- | --- |
    | 재고 차감 / 수량 감소 | 동시에 감소하면 음수, 초과 차감 발생 | Row Lock / Pessimistic Lock |
    | 좌석·인원 제한 (선착순) | 동시에 여러 명이 “마지막 1자리” 요청 | Pessimistic Lock 또는 Optimistic Lock |
    | 금액 이체 | 중간 상태 노출 시 치명적 | X-lock (Strict 2PL) |
    | 중복 방지 | 동시에 같은 데이터 INSERT | Unique Index + Lock |
    | 상태 전이 (결제/주문) | 같은 주문을 두 번 처리 방지 | Row Lock |
    | 배치/정산 | 읽는 중 데이터 변경 방지 | Gap / Next-Key Lock |
    | AUTO_INCREMENT | 중복 없는 PK 생성 | Auto Increment Lock (자동) |

## MVCC

> 데이터베이스가 동시에 여러 트랜잭션을 처리할 때, 각 트랜잭션이 자신만의 일관된 데이터 뷰를 가질 수 있도록 하는 기술
> 

어떻게? 데이터를 한 개만 두지 않고, 여러 `version`으로 관리한다!

<aside>
❓

트랜잭션 A가 SELECT → 동시에 트랜잭션 B가 UPDATE

**Problem :** 이떄, MVCC가 없다면, A는 B가 끝날 때까지 기다린다. 읽기 때문에 전체 서비스가 느려지고, 병목 현상이 발생

**Solution :** 읽는 사람은 과거 버전을 보게 하자! 
즉, 읽고 있는 사람은 기존 책을 보게 하고, 새로 들어온 사람은 새 책을 보게 하자!

</aside>

- Undo Log : 이전 버전 데이터 저장소
- 트랜잭션 스냅샷 : “내가 볼 시점” 기준

`UPDATE accounts SET balance = 15000 WHERE account_id = 1;`

- 위에 코드가 수행될때, 새 값 (15000)은 테이블에 바로 기록되고, 이전 값(10000)은 Undo Log에 저장된다.
- 따라서 각 트랜잭션은 적절한 버전을 선택해서 읽을 수 있다!

**터미널 1 - 트랜잭션 A**

```sql
START TRANSACTION;
SELECT balance FROM accounts; -- 10000
```

**터미널 2 - 트랜잭션 B**

```sql
UPDATE accounts SET balance = 15000;
COMMIT;
```

**터미널 1로 돌아와서**

```sql
SELECT balance FROM accounts; -- 여전히 10000
```

### 격리 수준에 따른 MVCC

1. `READ COMMITTED` - 매 쿼리마다 최신 커밋된 버전
    
    ```sql
    START TRANSACTION;
    SELECT balance; -- 15000
    -- 다른 트랜잭션 UPDATE + COMMIT
    SELECT balance; -- 20000 (바뀜)
    ```
    
    - 같은 트랜잭션이어도
    - SELECT할 때마다 **새 스냅샷**
2. `REPEATABLE READ` - 트랜잭션 시작 시점의 스냅샷 고정
    
    ```sql
    START TRANSACTION;
    SELECT balance; -- 20000
    -- 다른 트랜잭션 UPDATE + COMMIT
    SELECT balance; -- 여전히 20000
    ```
    

## 무결성 제약 조건

| **제약 조건** | **영문** | **적용 대상** | **핵심 내용** | **예시** |
| --- | --- | --- | --- | --- |
| **도메인 제약조건** | Domain Constraint | 속성(Attribute) | 속성 값은 **원자값**이어야 하며,  **허용 범위·타입·기본값**을 만족해야 함 | 나이 ≥ 0  성별 ∈ {M, F} |
| **키 제약조건** | Key Constraint | 키 속성 | **키 속성 값은 중복 불가**  (유일성 보장) | 학번, 사번 |
| **엔티티 무결성 제약조건** | Entity Integrity Constraint | 엔티티의 기본 키 | **기본 키는 NULL 불가**  엔티티를 유일하게 식별해야 함 | 학생(id PK)에서 id는 NULL  |
| **참조 무결성 제약조건** | Referential Integrity Constraint | 외래 키(FK) | 외래 키 값은  ① 참조하는 기본 키 값이거나  ② NULL 이어야 함 |  |

## 조인

| **JOIN 종류** | **기준** | **결과 개념** | **특징 요약** | **NULL 발생** |
| --- | --- | --- | --- | --- |
| **INNER JOIN** | 양쪽 테이블 | **교집합** | 두 테이블에 **모두 존재하는 행만 조회** | ❌ |
| **LEFT OUTER JOIN** | 왼쪽 테이블 | 왼쪽 기준 + 교집합 | 왼쪽 테이블은 **전부 유지**,  오른쪽은 매칭되는 값만 | ⭕ (오른쪽) |
| **RIGHT OUTER JOIN** | 오른쪽 테이블 | 오른쪽 기준 + 교집합 | 오른쪽 테이블은 **전부 유지**,  왼쪽은 매칭되는 값만 | ⭕ (왼쪽) |
| **FULL OUTER JOIN** | 없음 | **합집합** | 양쪽 테이블 **모든 행 조회** | ⭕ (양쪽) |
| **CROSS JOIN** | 없음 | 데카르트 곱 | 모든 조합 생성  결과 개수 = **N × M** | ❌ |
| **SELF JOIN** | 자기 자신 | 관계에 따라 다름 | **같은 테이블을 JOIN**  (부모–자식, 상사–부하 등) | 조건에 따라 |
