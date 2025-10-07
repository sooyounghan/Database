-----
### DDL - 테이블 변경, 제거
-----
1. ALTER TABLE : 이미 만든 테이블의 구조 변경
   - 테이블의 구조를 변경해야 할 때 사용
   - 예를 들어, 고객에게 포인트를 지급하는 제도를 새로 도입해서 customers 테이블에 point 열을 추가해야 할 수 있음
   - 이럴 때 사용하는 것이 ALTER TABLE 명령어

2. 열(Column) 추가하기 - ADD COLUMN
   - customers 테이블에 point 열을 추가
```sql
ALTER TABLE customers
ADD COLUMN point INT NOT NULL DEFAULT 0;
```
   - DESC customers 로 확인해보면 point 열이 추가된 것을 볼 수 있음

3. 열(Column) 데이터 타입 변경하기 - MODIFY COLUMN
```sql
ALTER TABLE customers
MODIFY COLUMN address VARCHAR(500) NOT NULL;
```
   - DESC customers 로 확인해보면 address 컬럼의 길이가 500으로 변경된 것을 확인할 수 있음

4. 열(Column) 삭제하기 - DROP COLUMN
```sql
ALTER TABLE customers
DROP COLUMN point;
```
   - DESC customers 로 확인해보면 point 컬럼이 제거된 것을 확인할 수 있음

5. 실무 경고
   - ALTER TABLE 은 유용한 기능이지만, 조심해서 사용해야 함
   - 수백만, 수천만 건의 데이터가 들어있는 거대한 테이블의 구조를 변경하는 작업은 엄청나게 많은 시간과 시스템 자원을 소모
   - 작업 중에는 테이블이 잠겨서 서비스가 일시적으로 멈출 수도 있음
   - 따라서 실무에서는 사용자가 적은 새벽 시간을 이용해 점검 시간에 맞춰 작업하는 것이 일반적
   - 참고로 최신 버전의 MySQL에서는 열을 추가하는 것 정도는 실시간으로 적용해도 크게 문제가 되지는 않음
   - 그래도 대부분의 라이브 변경 작업은 데이터베이스 버전에 맞는 매뉴얼을 확인한 다음에 어느정도 잠금이 발생하는지, 라이브 상황에 수행할 수 있는지 확인한 다음에 실행하는 것을 권장

6. DROP TABLE vs TRUNCATE TABLE
   - 테이블을 다루다 보면 테이블의 내용을 비워야 할 때가 있음
   - 이때 DROP 과 TRUNCATE 는 비슷해 보이지만 아주 큰 차이가 존재
   - DROP TABLE : 테이블의 존재 자체를 삭제
     + DROP TABLE orders; 를 실행하면, orders 테이블의 모든 데이터는 물론, orders라는 테이블의 구조(설계도)까지 완전히 사라짐
     + 테이블을 다시 사용하려면 CREATE TABLE부터 다시 해야 함
   - TRUNCATE TABLE : 테이블의 구조는 남기고, 내부 데이터만 모두 삭제
     + TRUNCATE TABLE orders; 를 실행하면, orders 테이블 안의 모든 주문 데이터가 순식간에 사라짐
     + 하지만 orders 라는 테이블의 구조(열, 제약조건 등)는 그대로 남아있어서, 바로 새로운 데이터를 INSERT 할 수 있음
   - 특징
     + DELETE FROM orders; (WHERE 절 없는 DELETE)와 결과적으로는 같아 보이지만, TRUNCATE가 훨씬 빠름
     + DELETE 는 한 줄씩 지우면서 삭제 기록을 남기는 반면, TRUNCATE 는 테이블을 초기화하는 개념이라 내부 처리 방식이 더 간단하고 빠름
     + TRUNCATE는 AUTO_INCREMENT 값도 초기화
     + 만약 orders 테이블에 1000개의 주문이 있어서 다음 order_id 가 1001일 차례였다면, TRUNCATE 이후에는 다시 1부터 시작 (DELETE는 AUTO_INCREMENT 값을 초기화하지 않음)

   - 이 둘의 차이를 명확히 이해하고 상황에 맞게 사용하는 것은 매우 중요
   - 테스트 데이터를 모두 지우고 처음부터 다시 시작하고 싶을 때는 TRUNCATE가 유용하고, 테이블 자체가 더 이상 필요 없을 때는 DROP을 사용

7. 제약 조건 무시하기
   - products 테이블은 orders에서 참조하고 있음
   - 이렇게 참조 당하는 테이블은 DROP, TRUNCATE를 할 수 없음
```sql
DROP TABLE products;
```
   - 실행 결과
```
Error Code: 3730. Cannot drop table 'products' referenced by a foreign key
constraint 'fk_orders_products' on table 'orders'.
```
   - 실행 결과를 보면 orders 테이블에 있는 fk_orders_products 외래 키 제약 조건 때문에 테이블을 제거할 수 없다는 뜻
   - 만약 이 테이블을 제거할 수 있게 둔다면 orders 테이블 입장에서는 주문된 상품의 정보를 찾을 수 없는(상품 없는 주문) 심각한 문제가 나타남
```sql
TRUNCATE TABLE products;
```
   - 실행 결과
```
Error Code: 1701. Cannot truncate a table referenced in a foreign key
constraint (`my_shop`.`orders`, CONSTRAINT `fk_orders_products`)
```
   - TRUNCATE 의 경우도 마찬가지로, TRUNCATE 는 내부의 데이터를 확인하지 않고, 모든 데이터를 빠르게 제거
   - 따라서 이 테이블의 모든 데이터를 제거한다면 orders 테이블 입장에서는 주문된 상품의 정보를 찾을 수 없는(상품 없는 주문) 심각한 문제가 발생할 수 있음
   - 만약 이런 외래 키 제약조건을 임시로 무시하고 싶다면 외래 키 체크를 비활성화 하면 됨
```sql
SET FOREIGN_KEY_CHECKS = 0; -- 비활성화

SET FOREIGN_KEY_CHECKS = 1; -- 활성화
```
```sql
SET FOREIGN_KEY_CHECKS = 0; -- 비활성화
TRUNCATE TABLE products;

SET FOREIGN_KEY_CHECKS = 1; -- 활성화
```

8. 주의사항
   - FOREIGN_KEY_CHECKS를 비활성화하면 데이터 무결성이 깨질 수 있으므로, 필요한 작업을 마친 후에는 반드시 다시 활성화해야 함
   - SET XXX 방식의 설정은 데이터베이스 접속이 연결되는 동안만 유효하며, 연결이 끊기면 설정이 사라지므로, 따라서 다시 접속하는 경우 설정을 다시 해야 함
  
