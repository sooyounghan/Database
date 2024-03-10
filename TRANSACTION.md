-----
### 트랜잭션 (Transaction)
-----
1. 최종 결과를 내기까지 위한 하나의 작업 단위를 의미
2. 개발자가 전달한 INSERT / UPDATE / DELETE 문을 메모리상에서만 수행하고, 디스크에는 미반영
   - TABLE과 관련된 작업은 바로 디스크에 반영
3. 이는 실수로 인한 데이터 유실을 막기 위함
4. 따라서, 데이터베이스를 조작하는 작업이 완료되고, 모두 정상적으로 되었다면, 이를 디스크에 반영
5. 작업이 시작되고 디스크에 반영될 때까지의 작업의 단위
6. 트랜잭션이 완료되면 디스크에 반영되어 저장

-----
### 트랜잭션 (Transaction) 명령어
-----
1. COMMIT : 트랜잭션을 완료하고 디스크에 마지막으로 반영 (복구 불가)
```sql
COMMIT [WORK] [TO [SAVEPOINT] 세이브포인트명];
```
  - WORK는 가독성 향상 목적으로 사용하나 주로 생략

2. ROLLBACK : 트랜잭션을 취소, DB에 가해진 변경사항을 취소
```sql
ROLLBACK [WORK] [TO [SAVEPOINT] 세이브포인트명];
```

3. SAVEPOINT
  - 작업 전체 취소가 아닌 특정 부분에서 트랜잭션을 취소
  - 즉, ROLLBACK 단위를 지정
```sql
SAVEPOINT 세이브포인트명;
```

4. TRUNCATE
   - DELETE문은 COMMIT을 실행해야 데이터가 완전히 삭제 (즉, ROLLBACK하면 삭제 전으로 복귀)
   - TRUNCATE문은 한 번 실행하면 데이터가 바로 삭제되고, ROLLBACK 되지 않음
   - WHERE 조건을 붙일 수 없고, 테이블 전체가 바로 삭제
```sql
TRUNCATE TABLE [스키마명.]테이블명;
```
