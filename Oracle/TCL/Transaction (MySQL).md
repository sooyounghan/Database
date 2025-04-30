-----
### 트랜잭션 (Transaction)
-----
1. 데이터베이스에서 데이터 처리의 한 단위
2. 대부분의 데이터베이스는 데이터를 저장하고 수정하고 삭제하는 작업을 바로 물리적인 하드디스크에 저장된 데이터에 반영하지 않음
  - 이는 개발자의 실수로 잘못된 명령문을 입력했을 경우 다시 원래 상태로 되돌리기 위한 안전 장치
3. 따라서, 개발자는 Commit이라는 작업을 하기 전까지 입력한 명령문은 메모리에서만 동작하고, 물리적 하드디스크에 반영하지 않으며, Commit 작업이 발생하면 그 때 하드디스크에 반영
4. 개발자가 데이터에 대한 작업을 하기 위해 입력하는 명령문들의 시작부터 Commit까지를 하나의 트랜잭션이라 함

-----
### ROLLBACK
-----
1. 데이터를 저장, 삭제, 수정 등의 작업을 하고 난 후 원래의 형태로 되돌리는 작업
2. Commit이라는 작업을 하고 난 이후에는 RollBack 작업을 해도 원래 형태로 되돌릴 수 없음
3. WorkBench 등의 프로그램에서는 자동으로 Commit 작업이 발생하므로 RollBack을 해도 원래 형태로 되돌릴 수 없음
4. 설정 변경 방법 : Edit - Preferences - SQL Editor - SQL Execution - New connections use auto commit mode 해제
  - SET AUTOCOMMIT = 1 (AutoCommit 활성화)
  - SET AUTOCOMMIT = 0 (AutoCommit 비활성화)
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/4ba9100a-cd8d-4c81-b058-76f507cb77f8">
</div>

4. 예제
   - TEST_TABLE2의 모든 데이터 삭제 후, ROLLBACK 실행 시, DELETE문으로 삭제된 데이터가 복원된 것 확인 가능
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/e33127db-c225-4d14-afb9-5e1886a88bd8">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/f714b6c9-4f60-4b09-80c7-47a7d142fcc8">
</div>

-----
### COMMIT
-----
1. 하나의 트랜잭션을 물리적인 데이터베이스에 적용하는 작업
2. 이 작업을 하게 되면, RollBack을 해도 되돌릴 수 없음
3. 일반적으로 개발자가 만드는 프로그램들은 자동으로 Commit이 발생
4. 예제
   - TEST_TABLE2의 모든 데이터 삭제 후, Commit 실행 시, DELETE 문으로 삭제된 모든 데이터가 반영되어 영구 삭제가 됨
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/65e70617-cc06-4785-87a4-de3b8749b422">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/3609575e-75b6-42f0-ad3c-1c126ad4d458">
</div>

-----
### SAVEPOINT
-----
1. Save Point를 지정하면, RollBack 시 지정된 위치로 복원 가능
2. SAVEPOINT 명령어로 지점을 지정하고 ROLLBACK 명령어로 복원
```sql
SAVEPOINT 이름;
```

```sql
ROLLBACK; // 이전 Commit을 최종적으로 한 시점으로 이동
ROLLBACK TO SAVEPOINT명; // SAVEPOINT 지점으로 이동
```

4. 예제
   - TEST_TABLE2에서 DATA1이 100인 값에 대해 DATA2와 DATA3의 값을 변경
   - TEST_TABLE2에서 DATA1이 100인 데이터를 삭제
   - 이 시점에서 Rollback을 하면, 이전 Commit 지점으로 RollBack
   - UPDATE문을 실행 후, SAVEPOINT S1을 설정하면, RollBack 시점이 SAVEPOINT S1을 시점으로 이동
  
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/1fbbb677-ef86-4579-8f08-3662bffeac72">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/382f8823-a5e7-4de2-935c-39c0d2a92768">
</div>

-----
### TRUNCATE
-----
1. TRUNCATE를 사용하면, 지정된 테이블의 모든 ROW를 삭제
2. DELETE문과 다른 점은 DELETE문은 데이터베이스에 바로 반영되지 않아 ROLLBACK이 가능
3. TRUNCATE는 바로 데이터베이스에 반영하므로 ROLLBACK이 불가능함
```sql
TRUNCATE 테이블명;
```
 
4. 예제
   - 현 시점에서 COMMIT 후 DELETE문을 실행하면, 모든 데이터가 삭제되었지만, ROLLBACK을 하면 COMMIT 시점으로 복원
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/bf17058f-a947-465f-8689-881dc20d8f22">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/11c92026-5f90-4bb5-a00f-e7577b93045e">
</div>

   - 하지만, TRUNCATE를 실행하면, ROLLBACK해도 반영되지 않음을 알 수 있음
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/22e5e691-8a35-4c91-9251-09de43786da0">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/f8dc87d0-fbf0-4b58-a2e5-13297d0fb17e">
</div>
