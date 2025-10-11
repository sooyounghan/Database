-----
### 정리
-----
1. 트랜잭션이 필요한 이유
   - 데이터 변경 시 여러 작업을 논리적으로 쪼갤 수 없는 하나의 묶음(Unit of Work)으로 다루기 위해 트랜잭션이 필요
   - 주문 생성(INSERT)과 재고 감소(UPDATE)처럼 두 작업 중 하나만 성공하면 데이터 일관성이 깨지는 심각한 문제가 발생
   - 트랜잭션은 '전부 아니면 전무(All or Nothing)' 원칙을 보장
   - 묶인 작업 중 하나라도 실패하면, 이전에 성공한 모든 작업을 자동으로 취소(원상 복구)하여 데이터의 일관성과 안정성을 지킴

2. 커밋, 롤백
   - 트랜잭션은 START TRANSACTION, COMMIT, ROLLBACK 명령어로 제어
     + START TRANSACTION : 트랜잭션의 시작을 데이터베이스에 알림
     + COMMIT : 트랜잭션 내의 모든 작업이 성공했을 때, 변경 사항을 영구적으로 저장
     + ROLLBACK : 문제가 발생했을 때, 트랜잭션 내에서 실행한 모든 변경 사항을 취소하고 시작 전 상태로 되돌림
   - MySQL은 기본적으로 autocommit 모드가 활성화되어 있어, 모든 SQL 문이 즉시 커밋되므로 여러 문장을 묶으려면 START TRANSACTION을 명시적으로 사용해야 함

3. 트랜잭션의 ACID 속성
   - 신뢰할 수 있는 트랜잭션은 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)이라는 ACID 속성을 반드시 만족해야 함
   - 원자성 (Atomicity) : 트랜잭션은 전부 성공하거나 전부 실패해야 함 (All or Nothing)
   - 일관성 (Consistency) : 트랜잭션 완료 후에도 데이터베이스는 제약 조건 등 유효한 상태를 유지해야 함
   - 격리성 (Isolation) : 여러 트랜잭션이 동시에 실행될 때, 서로의 작업 중간 결과에 간섭할 수 없음
   - 지속성 (Durability) : 성공적으로 COMMIT 된 트랜잭션의 결과는 시스템 장애가 발생해도 영구적으로 보존

4. 트랜잭션 격리 수준
   - 격리 수준은 데이터 정합성과 동시성(성능) 사이의 트레이드오프를 조절하는 설정
   - 격리 수준이 낮으면 동시성 문제는 발생할 수 있으나 성능이 향상되고, 높이면 정합성은 보장되나 성능이 저하
   - 동시성 문제 유형 : 더티 리드(Dirty Read), 반복 불가능 읽기(Non-Repeatable Read), 유령 읽기(Phantom Read) 등이 있음
   - 표준 격리 수준: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE 순으로 격리 강도가 높아짐
   - MySQL InnoDB 엔진의 기본 격리 수준은 REPEATABLE READ이며, 대부분의 비즈니스 환경에서 정합성과 성능의 합리적인 균형을 제공하므로 특별한 이유가 없다면 기본값을 사용하면 됨
     + 트래픽이 매우 많고 반복 불가능 읽기, 유령 읽기 상황이 자주 발생하지 않는 웹 애플리케이션이라면 READ COMMITTED을 고려할 수 있음
