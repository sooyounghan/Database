-----
### 외래 키 제약 조건
-----
1. 하지만 데이터베이스의 진정한 힘은 '관계'에 있고, 이 관계는 여러 테이블에 걸쳐서 맺어짐
   - 여기서 가장 중요한 무결성 원칙, 참조 무결성(Referential Integrity)이 등장
   - 참조 무결성 : 두 테이블의 관계가 항상 유효하고 일관된 상태를 유지해야 한다는 원칙
   - 예를 들어, orders 테이블의 user_id 컬럼은 반드시 users 테이블에 실제로 존재하는 user_id 값만을 참조 해야 함 (만약 존재하지도 않는 유령 회원의 주문이 있다면, 이 관계는 깨진 것)
   - 이러한 테이블 간의 관계 무결성을 강제하는 가장 강력한 제약 조건이 바로 외래 키(Foreign Key)

2. 외래 키(FK)의 역할 : 유령 데이터를 막기
   - 외래 키 제약 조건은 두 가지 중요한 규칙을 통해 우리의 데이터를 보호
     + 자식 테이블(orders)에 INSERT, UPDATE 할 때 : 부모 테이블(users)에 존재하지 않는 user_id 값을 자식 테이블(orders)의 user_id 컬럼에 넣으려는 시도를 막음 (유령 주문 생성 방지)
     + 부모 테이블(users)에서 DELETE, UPDATE 할 때: 자식 테이블(orders)에서 참조하고 있는 user_id 값을 가진 행을 함부로 삭제하거나 변경하지 못하게 막음 (기존 주문을 유령 주문으로 만드는 것 방지)

   - 위반 시나리오 : 우리 테이블에는 이미 외래 키가 설정되어 있는데, 이 규칙을 직접 위반
     + 유령 주문 생성 시도 : 존재하지 않는 user_id 인 999번 고객의 주문을 넣어보기
```sql
-- 존재하지 않는 user_id(999)로 주문을 생성 시도 (에러 발생)
INSERT INTO orders (user_id, product_id, quantity)
VALUES (999, 1, 1);
```
   - 실행 결과 에러
```
Error Code: 1452. Cannot add or update a child row: a foreign key constraint
fails (`my_shop2`.`orders`, CONSTRAINT `fk_orders_users` FOREIGN KEY
(`user_id`) REFERENCES `users` (`user_id`))
```
   - 데이터베이스는 fk_orders_users 외래 키 제약 조건이 실패했다는 명확한 에러를 출력하며, 이 INSERT 를 거부

   - 주문이 있는 고객 삭제 시도 : '션' 고객(user_id=1)은 이미 주문 기록이 있음
      + orders 테이블의 '션' 고객(user_id =1)의 주문
```sql
SELECT *
FROM orders
WHERE user_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/cec50fd7-8650-4ca1-b3f2-2faf5d835893">
</div>

   - 이 고객을 users 테이블에서 삭제하려고 시도
```sql
-- 자식 테이블(orders)에서 참조하고 있는 부모 데이터(users)를 삭제 시도 (에러 발생)
DELETE
FROM users
WHERE user_id = 1;
```
   - 실행 결과 에러
```
Error Code: 1451. Cannot delete or update a parent row: a foreign key
constraint fails (`my_shop2`.`orders`, CONSTRAINT `fk_orders_users` FOREIGN
KEY (`user_id`) REFERENCES `users` (`user_id`))
```

  - 역시나 데이터베이스는 이 고객을 참조하는 주문이 있기 때문에 삭제할 수 없다는 오류를 발생시키며 삭제를 막음
   - 만약 이 삭제 쿼리가 성공한다면 orders 테이블에는 '션' 고객(user_id =1)의 주문이 남아있지만, users 테이블의 션은 제거
   - 결과적으로 주문 내역만 있고, 실제 고객은 사라지는 심각한 문제가 발생 : 그리고 향후 이 주문이 누구의 주문인지 찾을 수 없게 됨
   - 만약 부모 테이블의 션(user_id = 1) 데이터를 삭제하고 싶다면 다음과 같이 자식 테이블의 션 관련 데이터를 먼저 삭제하고, 이후에 부모 테이블을 삭제하는 순서로 진행해야 함
```sql
DELETE
FROM orders
WHERE user_id = 1; -- 1.자식 테이블의 션(user_id = 1) 관련 데이터 제거
```
```sql
DELETE
FROM users
WHERE user_id = 1; -- 2.부모 테이블의 션(user_id = 1) 관련 데이터 제거
```

3. 💡 ON DELETE / ON UPDATE 옵션
   - 데이터베이스가 부모 데이터의 삭제나 수정을 무조건 막는 것이 기본값(RESTRICT)이며 가장 안전한 정책
   - 하지만 때로는 비즈니스 규칙에 따라 다른 정책이 필요할 수 있음
     + 예를 들어 "회원이 탈퇴하면, 그 회원의 모든 주문 기록도 함께 삭제되어야 하는" 경우
     + 이럴 때 사용하는 것이 외래 키의 ON DELETE와 ON UPDATE 옵션
   - RESTRICT (기본값) : 자식 테이블에 참조하는 행이 있으면 부모 테이블의 행을 삭제/수정할 수 없음
   - CASCADE : 부모 테이블의 행이 삭제/수정되면, 그를 참조하는 자식 테이블의 행도 함께 자동으로 삭제/수정
   - SET NULL : 부모 테이블의 행이 삭제/수정되면, 자식 테이블의 해당 외래 키 컬럼의 값을 NULL로 설정 (단, 이 옵션을 쓰려면 자식 테이블의 외래 키 컬럼이 NULL을 허용해야 함)

4. ON DELETE CASCADE 실습
   - 기존 orders 테이블을 삭제하고, ON DELETE CASCADE 옵션을 추가하여 다시 생성
```sql
-- 실습을 위해 기존 테이블 삭제 후 CASCADE 옵션으로 재생성
DROP TABLE orders;
CREATE TABLE orders (
     order_id BIGINT AUTO_INCREMENT,
     user_id BIGINT NOT NULL,
     product_id BIGINT NOT NULL,
     order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
     quantity INT NOT NULL,
     status VARCHAR(50) DEFAULT 'PENDING',
     PRIMARY KEY (order_id),
     CONSTRAINT fk_orders_users FOREIGN KEY (user_id)
     REFERENCES users(user_id) ON DELETE CASCADE, -- CASCADE 옵션 추가
     CONSTRAINT fk_orders_products FOREIGN KEY (product_id)
     REFERENCES products(product_id)
);
```
```sql
-- 션 회원 다시 등록
INSERT INTO users(user_id, name, email, address, birth_date)
VALUES (1, '션', 'sean@example.com', '서울시 강남구', '1990-01-15');
```
```sql
-- 주문 데이터 다시 입력
INSERT INTO orders(user_id, product_id, quantity, status) VALUES
(1, 1, 1, 'COMPLETED'),
(1, 4, 2, 'COMPLETED'),
(2, 2, 1, 'SHIPPED');
```
   - 앞서 user_id = 1 션 회원을 삭제했다면, 스크립트를 참고해서 먼저 션 회원을 다시 등록
   - 만약 user_id = 1 션 회원을 삭제하지 않았다면 "션 회원 다시 등록" 스크립트는 실행하지 말 것
   - 새로 저장한 주문 데이터 확인
```sql
SELECT *
FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5eeaa66a-9574-419d-a4f2-abb931716198">
</div>

   - '션' 고객(user_id=1)의 주문이 2건 있는 것을 확인할 수 있음 (order_id : 1, 2)
   - 이제 다시, 주문 기록이 있는 '션' 고객(user_id =1)을 삭제
```sql
DELETE
FROM users
WHERE user_id = 1;
```
   - 이번에는 아무 에러 없이 쿼리가 성공적으로 실행
   - 전체 주문 기록을 조회
```sql
SELECT *
FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/02fdc7bf-c2c3-4967-a186-1769e91ca1d1">
</div>

   - users 테이블에서 user_id가 1번 고객을 삭제하자, 그를 참조하던 orders 테이블의 주문 기록들이 연쇄적으로 함께 삭제된 것을 확인할 수 있음 (order_id : 1, 2 제거)
   - CASCADE 옵션은 매우 편리하지만, 의도치 않은 대량의 데이터 삭제를 유발할 수 있으므로 반드시 비즈니스 로직을 명확히 이해하고 신중에 신중을 기해서 사용해야 함

5. CASCADE - 실무 이야기
   - CASCADE 옵션은 분명 편리한 기능이지만, 잘못 사용할 경우 예상치 못한 대량의 데이터가 함께 삭제되는 경우가 발생
   - 특히 관계가 복잡하게 얽혀 있는 경우에는 파급 효과를 예측하기 어려움
     + 이런 문제로 실무에서는 CASCADE 옵션은 잘 사용하지 않음
     + 대신에 애플리케이션 계층에서 명시적으로 관련된 데이터를 처리하는 방식이 더 선호

   - 외래 키는 테이블 간의 관계를 정의하는 것을 넘어, 그 관계가 항상 올바른 상태를 유지하도록 강제하는 데이터베이스의 핵심적인 무결성 장치
