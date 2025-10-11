-----
### 커밋, 롤백
-----
1. '전부 아니면 전무(All or Nothing)'라는 트랜잭션의 약속은 데이터베이스의 안정성을 보장하는 핵심 원리
2. 트랜잭션을 직접 제어하는 세 가지 핵심 명령어
   - START TRANSACTION : 지금부터 트랜잭션을 시작한다고 데이터베이스에 선언
     + 이 명령어 이후에 실행되는 모든 쿼리는 하나의 트랜잭션으로 묶여 관리됨 (BEGIN 이라고도 씀)

   - COMMIT : 트랜잭션 내의 모든 작업이 성공적으로 완료되었으니, 지금까지의 변경 사항을 디스크에 영구적으로 저장(반영)하라는 의미의 최종 승인 명령어
     + COMMIT 이 실행된 후에는 변경 내용을 되돌릴 수 없음
     
   - ROLLBACK : 문제가 발생하여 작업을 중단해라. 이 트랜잭션 내에서 실행했던 모든 변경 사항을 전부 취소하고, 트랜잭션이 시작되기 직전의 상태로 완벽하게 되돌려라라는 의미의 작업 취소(원상 복구) 명령어
     
3. 실습 : 계좌이체 시나리오
  - A 고객의 계좌에서 B 고객의 계좌로 10,000원을 이체하는 상황
  - accounts 테이블 생성
```sql
DROP TABLE IF EXISTS accounts;

CREATE TABLE accounts (
     id INT PRIMARY KEY,
     owner_name VARCHAR(255) NOT NULL,
     balance INT NOT NULL
);
```
```sql
-- 초기 데이터 입력
INSERT INTO accounts (id, owner_name, balance) VALUES
(1, 'A고객', 50000),
(2, 'B고객', 20000);
```

4. 시나리오 1 : 성공적인 계좌이체 (COMMIT)
   - 거래 전 잔고 확인 : 거래 전 두 고객의 잔고를 확인
```sql
SELECT *
FROM accounts;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/34d0df06-1ba1-406a-81a1-ffbcac088fb3">
</div>

   - 트랜잭션 시작 및 계좌이체 실행 : 트랜잭션을 시작하고, 두 개의 UPDATE 문을 차례로 실행
```sql
-- 트랜잭션 시작
START TRANSACTION;

-- 1번 작업 : A고객 계좌에서 10,000원 출금
UPDATE accounts
SET balance = balance - 10000
WHERE id = 1;

-- 2번 작업 : B고객 계좌에 10,000원 입금
UPDATE accounts
SET balance = balance + 10000
WHERE id = 2;
```

  - 최종 확정 (COMMIT) : 두 작업이 모두 성공적으로 실행되었음을 확인하고, COMMIT 명령어로 변경 사항을 데이터베이스에 영구적으로 반영
```sql
COMMIT;
```

  - 거래 후 최종 잔고 확인 : COMMIT 이후에 다시 잔고를 확인
```sql
SELECT *
FROM accounts;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f43c78e6-0b20-489b-91d5-48a38c143693">
</div>

  - 계좌의 잔고가 성공적으로 변경되어 영구 저장된 것을 확인할 수 있음

5. 시나리오 2: 실수로 인한 작업 취소 (ROLLBACK)
   - 이번에는 A고객이 B고객에게 5,000원을 추가로 이체하려다가, 실수를 깨닫고 작업을 취소하는 상황을 가정
   - 트랜잭션 시작 및 출금 실행
```sql
-- 새로운 트랜잭션 시작
START TRANSACTION;

-- 1번 작업 : A고객 계좌에서 5,000원 출금
UPDATE accounts
SET balance = balance - 5000
WHERE id = 1;
```
   - 이 시점에서 A고객의 잔고는 임시적으로 35,000원이 되었음
   - 하지만 입금 전, 송금할 사람이 B가 아니었다는 사실을 깨달았거나 또는 무언가 오류가 발생해서 더는 진행하면 안되는 상황이 발생
   - 현재 작업 상황을 확인
```sql
SELECT *
FROM accounts;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8fa0a49b-7445-4546-b63a-42619e102cf5">
</div>

   - balance - 5000 연산으로 A 고객은 임시로 35000원으로 변경

   - 작업 취소 (ROLLBACK) : 이때 ROLLBACK을 실행하여, 이번 트랜잭션에서 수행한 모든 작업을 없었던 일로 되돌림
```sql
ROLLBACK;
```

   - 원상 복구된 잔고 확인 : ROLLBACK 이후에 다시 잔고를 확인
```sql
SELECT *
FROM accounts;
```

<div align="center">
<img src="https://github.com/user-attachments/assets/a9d4f770-c280-4b36-a079-55177f28bd97">
</div>

   - A 고객의 잔고가 UPDATE 이전 상태인 40,000원으로 완벽하게 돌아온 것을 볼 수 있음
   - ROLLBACK 이 출금 작업을 깨끗이 취소시킨 것

6. 💡 실무 팁 : MySQL의 autocommit
   - MySQL은 기본적으로 autocommit 모드가 활성화(ON 또는 1)되어 있음 : 이것은 우리가 실행하는 모든 SQL 문 하나하나를 각각의 트랜잭션으로 간주하여, 성공적으로 실행되는 즉시 자동으로 COMMIT 해버린다는 의미
   - 우리가 위에서 한 것처럼 여러 개의 문장을 하나의 트랜잭션으로 묶고 싶다면, 반드시 START TRANSACTION 명령어로 명시적으로 트랜잭션의 시작을 알려야 함
   - START TRANSACTION 을 실행하면, 그 세션에서는 COMMIT 이나 ROLLBACK 을 만날 때까지 autocommit이 잠시 비활성화

7. 💡 세션 용어
   - 세션은 데이터베이스 클라이언트(예: MySQL Workbench, DBeaver, 또는 터미널의 mysql 클라이언트)가 데이터베이스 서버에 연결된 순간부터 연결을 종료할 때까지의 논리적인 연결 단위
   - 각 세션은 독립적인 작업 공간을 가지며, 한 세션에서 시작된 트랜잭션은 해당 세션 내에서만 유효
   - COMMIT과 ROLLBACK은 데이터의 일관성을 지키는 개발자의 가장 중요한 두 가지 도구
