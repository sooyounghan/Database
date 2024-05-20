-----
### 트랜잭션 (Transaction)
-----
1. 데이터베이스에서 데이터 처리의 한 단위
2. 대부분의 데이터베이스는 데이터를 저장하고 수정하고 삭제하는 작업을 바로 물리적인 하드디스크에 저장된 데이터에 반영하지 않음
  - 이는 개발자의 실수로 잘못된 명령문을 입력했을 경우 다시 원래 상태로 되돌리기 위한 안전 장치
3. 따라서, 개발자는 Commit이라는 작업을 하기 전까지 입력한 명령문은 메모리에서만 동작하고, 물리적 하드디스크에 반영하지 않으며, Commit 작업이 발생하면 그 때 하드디스크에 반영
4. 개발자가 데이터에 대한 작업을 하기 위해 입력하는 명령문들의 시작부터 Commit까지를 하나의 트랜잭션이라 함

-----
### RollBack
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
### Commit
-----
1. 하나의 트랜잭션을 물리적인 데이터베이스에 적용하는 작업
2. 이 작업을 하게 되면, RollBack을 해도 되돌릴 수 없음
3. 일반적으로 개발자가 만드는 프로그램들은 자동으로 Commit이 발생

