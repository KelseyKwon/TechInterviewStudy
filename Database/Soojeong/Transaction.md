이번엔 아래와 같은 내용을 정리하는게 스터디의 과제라 진행해보겠습니다. 

```
트랜잭션
트랜잭션 acid
트랜잭션 격리수준
commit
rollback
갱신 분실 문제
locking
스토리즈 엔진 락
mvcc(다중 버전 동시성 제어)과 락
조인
```

### 트랜잭션

S가 J에게 돈을 20만원 이체하려고 할 때, 

S

```sql
UPDATE account SET balance = balance - 200000 WHERE id = 'S';
```

J

```sql
UPDATE account SET balance = balance + 200000 WHERE id = 'J';
```

→ 두가지가 모두 성공할 때, 이체도 성공

트랜잭션 ?

- 단일한 논리적인 작업 단위 ( a single logical unit of work )
- 논리적인 이유로 여러 SQL문들을 단일 작업으로 묶어서 나눠질 수 없게 만든 것
- 트랜잭션의 sql문들 중에 일부만 성공해서 db에 반영되는 일은 일어나지 않음

S가 J에게 돈을 20만원 이체,

```sql
> select * from account;
id balance
s  1000000
j  2000000

> START TRANSACTION;
> UPDATE account SET balance = balance - 200000 WHERE id = 'S';
> UPDATE account SET balance = balance + 200000 WHERE id = 'J';
> COMMIT;

> select * from account;
id balance
s   800000
j  2200000
```

COMMIT

- 지금까지 작업한 내용을 db에 영구적으로 저장
- 트랜잭션을 종료

30만원 추가 이체하다가 취소하는 경우,

```sql
> select * from account;
id balance
s   800000
j  2200000

> START TRANSACTION;
> UPDATE account SET balance = balance - 300000 WHERE id = 'S';
> select * from account;
id balance
s   500000
j  2200000

> ROLLBACK;
```

ROLLBACK

- 지금까지 작업들을 모두 취소하고 트랜잭션 이전 상태로 되돌림
- 트랜잭션을 종료

```sql
> select * from account;
id balance
s   800000
j  2200000
```

```sql
> SELECT @@AUTOCOMMIT;
```

AUTOCOMMIT

- 각각의 sql문을 자동으로 트랜잭션 처리 해주는 개념
- sql문이 성공적으로 실행하면 자동으로 커밋
- 실행 중에 문제가 있었다면 알아서 롤백
- mysql에서는 default로 autocommit이 enabled
- 다른 dbms에서도 대부분 같은 기능 제공

```sql
> insert into account values ('W', 1000000);
```

- autocommit이 enabled된 상태
- insert문을 실행하면 자동으로 커밋
- account 테이블에 ('W', 1000000) 데이터가 영구적으로 저장

정리할 내용이 많아서, 트랜잭션에 대한 간단한 정리만 하고 ACID 정리를 해야 겠다. ACID도 사실 많이 보는 개념인 것 같아 이거저거 적고만 넘어가야겠다. 

---

### ACID

Atomicity, 원자성

- ALL or NOTHING
- 트랜잭션은 논리적으로 쪼개질 수 없는 작업 단위이기 때문에 내부의 sql문들이 모두 성공해야 함
- 중간에 sql문이 실패하면 지금까지의 작업을 모두 취소하여 아무 일도 없었던 것처럼 rollback이 가능
- commit 실행 시 db에 영구적으로 저장하는 것은 dbms가 담당하는 부분
- rollback 실행 시 이전 상태로 되돌리는 것도 dbms가 담당하는 부분
- 개발자는 언제 commit 하거나 rollback 할지를 챙겨야 함

Consistency, 일관성

- 트랜잭션은 db 상태를 consistent 상태에서 또 다른 consistent 상태로 바꿔줘야 함
- constraints, trigger 등을 통해 db에 정의된 rules을 트랜잭션이 위반했다면 rollback 해야 함
- 트랜잭션이 db에 정의된 rule을 위반했는지 dbms가 commit 전에 확인하고 알려줌
- 그 외에 application 관점에서 트랜잭션이 consistent하게 동작하는지 개발자가 챙겨야 함

Isolation, 고립성

- 여러 트랜잭션들이 동시에 실행될 때도 혼자 실행되는 것처럼 동작하게 만듦
- dbms는 여러 종류의 isolation level을 제공
- 개발자는 isolation level 중에 어떤 level로 transaction을 동작시킬지 설정 할 수 있음
- concurrency control의 주된 목표가 isolation

Durability, 영속성

- commit된 트랜잭션은 db에 영구적으로 저장됨
- db system에 문제가 생겨도 commit된 transaction은 db에 남아있음
- ‘영구적으로 저장한다’라고 할 때는 일반적으로 비휘발성 메모리에 저장함을 의미
- 기본적으로 트랜잭션의 durability는 dbms가 보장

---

### sql은 트랜잭션들이 독립적으로 실행되는 것처럼 하기 위한 격리수준을 제공하는데,

격리수준

- READ UNCOMMITED
- READ COMMITED
- REPEATABLE READ
- SERIALIZABLE

문제

- DIRTY READ
- UNREPEATABLE
- PHANTOM READ

가 있다. 

### READ UNCOMMITED

: 커밋되지 않은 데이터에도 접근할 수 있는 격리 수준

( 가장 낮은 격리 수준 )

A 사용자가 id = 3 인 새로운 데이터를 추가 

B 사용자가 id = 3 인 데이터를 읽음

A 작업이 이때 커밋 ! 데이터 반영 !

→ 커밋되지 않은 데이터를 읽어도 문제가 없을까 ?

이때, 어떤 트랜잭션에서 처리한 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있는 현상인 Dirty Read 문제가 발생

만약, A가 커밋이 아닌 롤백을 했다면 ?

B는 잘못된 데이터로 처리를 하게 되는 문제가 발생함

Dirty Read 문제를 해결한 격리수준이,

### READ COMMITED

: 커밋된 데이터에만 접근할 수 있는 격리 수준

A 사용자가 id = 2 데이터 amy → brown 으로 업데이트

- undo log ?
    
    : 이전 상태의 데이터를 저장하는 곳
    
    amy는 undo log에 남고 brown은 데이터에 있음
    

B 사용자가 id = 2 인 데이터를 읽음

- B 사용자는 커밋된 기록만을 볼 수 있으므로, 
undo log에 있는 amy를 읽게 됨

A 작업이 이때 커밋 ! 데이터 반영 ! 다른 트랜잭션들이 조회 !

→ 커밋된 데이터만 읽으니까 이제 안전할까 ?

NON-REPEATABLE READ 문제가 발생

: 같은 트랜잭션 내에서 같은 행을 반복해서 읽었을 때
다른 트랜잭션이 그 행을 수정하거나 삭제하여 결과가 달라지는 현상

이번엔, B 사용자가 id = 6 인 데이터 soy를 가져옴

A 사용자가 id = 6 인 데이터 soy → chaehyun 으로 업데이트

A 사용자가 커밋 ! 실제 데이터베이스에는 chaehyun 삽입 !

이후, B 사용자가 id = 6 인 데이터 chaehyun을 가져옴

→ 같은 트랜잭션 내에서, 동일한 행의 데이터가 다르게 조회되는 문제 발생

READ COMMITED 격리 수준에서는 NON-REPEATABLE READ 라는 문제가 발생 

NON-REPEATABLE READ를 해결한 것이,

### REPEATABLE-READ

: 동일한 행을 여러 번 읽어도 항상 같은 결과를 보장하는 트랜잭션 격리 수준

( MySQL에서는 REPEATABLE-READ를 default 격리 수준으로 채택 )

B 사용자가 트랜잭션을 시작

TID( 트랜잭션 id )가 순차적으로 증가

+ 데이터 행에는 변경을 발생시킨 트랜잭션 번호가 포함

id = 6 인 데이터를 조회하면 chaehyun을 가져옴 

A 사용자가 트랜잭션을 시작 ( B 사용자의 TID보다 높은 ID를 가짐 )

A 사용자가 id = 6 인 데이터를 soojeong로 업데이트 

A 사용자 커밋 !

이때, B 사용자가 id = 6 인 데이터를 읽게 되면, chaehyun을 가져오게 됨.

→ B 트랜잭션의 id가 A가 커밋한 트랜잭션의 id보다 낮음

→ 자신의 트랜잭션 번호보다 작은 트랜잭션 번호에서 변경한 것만 보게 됨

→ 그래서 soojeong이 아닌 chaehyun을 가져오게 되는 것

NON-REPEATABLE 문제가 해결되었으니 안전할까 ?

일반적인 DBMS는 SELECT FOR UPDATE가 사용되는 경우 문제가 발생

먼저 트랜잭션을 시작한 B 사용자가 
‘SELECT FOR UPDATE’ → ‘나 이 데이터 변경할거니까 건들지마!’

→ 선택한 행에 잠금을 걸어 다른 트랜잭션이 해당 데이터에 접근하지 못하도록

이때, A 사용자가 새로운 값을 insert하게 되면

SELECT FOR UPDATE는 새로운 데이터 삽입까지 막진 못함

→ A 사용자의 insert 성공 !

B 사용자가 아까 읽었던 데이터에 방금 A 사용자가 insert한 데이터가 함께 읽혀옴

→ 귀신처럼 데이터 있다 없다 함 

→ PHANTOM READ 발생

: 같은 트랜잭션 내에서 동일한 쿼리를 반복 실행했을 때
다른 트랜잭션에 의해 새로운 데이터 행이 삽입되거나, 기존 데이터 행이 삭제되는 현상

‘MySQL은 REPEATABLE READ 격리 수준에서 PHANTOM READ를 방지’

MySQL에서는 검색 조건 범위에 해당되는 모든 인덱스 행에 잠금을 걺

→ 새로운 데이터 삽입을 방지 

→ A 사용자가 데이터 insert를 하려 하지만, 삽입하려는 곳에 잠금이 걸려 있어 대기 

→ B 사용자는 같은 결과를 보게 되는 좋은 ~ 결과 !

가장 높은 격리 수준인,

### SERIALIZABLE

: 한 트랜잭션에서 읽고 쓰는 레코드를 다른 트랜잭션에서는 절대 접근할 수 없는 트랜잭션 격리 수준

- 모든 읽기와 쓰기에 잠금을 건다
    
    → 동시성이 떨어짐
    
    → 이전 문제들은 발생하지 않음
    

---

### 갱신 분실 문제 (Lost Update)

: 두 개 이상의 트랜잭션이 같은 데이터를 읽고 수정할 때

한 트랜잭션의 수정 내용이 다른 트랜잭션에 의해 덮어써져 사라지는 현상

초기 데이터

`balance = 100`

### T1

```sql
SELECT balance FROM account WHERE id = 1; -- 100
```

### T2

```sql
SELECT balance FROM account WHERE id = 1; -- 100
```

---

### T1

```sql
UPDATE account SET balance = 110 WHERE id = 1;
```

### T2

```sql
UPDATE account SET balance = 120 WHERE id = 1;
```

최종 결과

`balance = 120`

→ T1의 + 10은 사라짐

→ 이게 갱신 분실 문제

어느 격리수준에서 발생할까 ?

- READ COMMITTED 에서 발생 가능
- REPEATABLE READ 이상에서는 보통 방지
- SELECT ... FOR UPDATE 사용하면 확실히 방지 가능

---

# locking

동시성 제어를 위해 DBMS가 사용하는 대표적인 방법

트랜잭션이 데이터를 읽거나 쓸 때 잠금(Lock)을 걸어

다른 트랜잭션의 접근을 제어함

**Shared Lock (S Lock)**

: 읽기 잠금

- 여러 트랜잭션이 동시에 가질 수 있음
- 읽기만 가능
- 쓰기는 불가능

**Exclusive Lock (X Lock)**

: 쓰기 잠금

- 하나의 트랜잭션만 가질 수 있음
- 읽기/쓰기 모두 다른 트랜잭션 차단

**SELECT FOR UPDATE**

```sql
SELECT * FROM account WHERE id = 1 FOR UPDATE;
```

→ 해당 행에 Exclusive Lock을 건다

→ 다른 트랜잭션은 해당 행 수정 불가

---

### 스토리지 엔진 락

MySQL은 스토리지 엔진에 따라 락 방식이 다름

대표적으로

- MyISAM
- InnoDB

**MyISAM**

- 테이블 단위 락 (Table Lock)
- 하나가 쓰기 하면 다른 트랜잭션은 대기
- 동시성 낮음

**InnoDB**

- 행 단위 락 (Row Lock)
- 필요 범위만 잠금
- 동시성 높음
- MVCC 지원

→ 실무에서 대부분 InnoDB 사용

**InnoDB의 락 종류**

- Record Lock
    
    특정 행 하나에 대한 락
    
- Gap Lock
    
    행과 행 사이의 "범위"에 대한 락
    
    → 새로운 insert 방지
    
- Next-Key Lock
    
    Record Lock + Gap Lock
    
    → REPEATABLE READ에서 Phantom Read 방지
    

---

### MVCC (Multi-Version Concurrency Control)

: 다중 버전 동시성 제어

락을 많이 사용하면 동시성이 떨어짐

그래서 등장한 개념이 MVCC

- 데이터가 수정될 때 이전 버전은 undo log에 저장
- 각 트랜잭션은 자신만의 snapshot을 가짐
- 읽기 시점에 맞는 버전을 선택

**동작 원리 간단 정리**

트랜잭션 시작

→ snapshot 생성

→ 이후 SELECT는 그 snapshot 기준으로 읽음

→ 다른 트랜잭션이 커밋해도 보이지 않음

장점

- 읽기 시 락이 거의 필요 없음
- 읽기와 쓰기 충돌 감소
- 성능 향상

MVCC와 Lock의 관계

- 일반 SELECT → MVCC 사용 (락 거의 없음)
- SELECT FOR UPDATE → Lock 사용
- UPDATE / DELETE → Lock 사용

즉,

읽기 → MVCC

쓰기 → Lock

---

### 조인 (JOIN)

둘 이상의 테이블을 연결하여 데이터를 조회하는 방법

**INNER JOIN**

```sql
SELECT * FROM A INNER JOIN B ON A.id = B.id;
```

→ 양쪽 테이블 모두에 존재하는 데이터만 조회

**LEFT OUTER JOIN**

```sql
SELECT * FROM A LEFT JOIN B ON A.id = B.id;
```

→ A는 모두 조회

→ B는 매칭되는 것만

**RIGHT OUTER JOIN**

→ LEFT JOIN의 반대

**FULL OUTER JOIN**

→ 양쪽 모두 포함

(MySQL은 기본적으로 지원하지 않음, UNION으로 구현)

조인의 실행 방식

**Nested Loop Join**

- 한 행씩 비교
- 인덱스 있으면 효율적

**Hash Join**

- 메모리에 해시 테이블 생성 후 조인
- 대용량에서 효율적

**Sort Merge Join**

- 정렬 후 병합