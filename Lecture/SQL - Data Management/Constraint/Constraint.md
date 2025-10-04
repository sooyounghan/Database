-----
### 제약 조건 활용
-----
1. 테이블을 만들 때 설정한 제약 조건(Constraints)은 실수로 혹은 의도적으로 잘못된 데이터가 들어오는 것을 막는 강력한 수문장 역할
2. 제약 조건에 위배되면, 데이터베이스는 단호하게 입력을 거부하며 오류를 발생
3. 예제를 실행하기 전에 먼저 기존 테이블을 초기화
```sql
SET FOREIGN_KEY_CHECKS = 0; -- 비활성화

truncate table products;
truncate table customers;
truncate table orders;

SET FOREIGN_KEY_CHECKS = 1; -- 활성화
```

4. NOT NULL 제약 조건 위반 : "필수 항목을 입력해주세요."
   - 가장 기본적이고 단순한 제약 조건은 NOT NULL
   - NULL 은 값이 없다는 뜻으로, NOT NULL은 'NULL 을 허용하지 않는다'는 뜻으로, 필수 입력 항목을 지정할 때 사용
   - customers 테이블의 name 열은 NOT NULL로 설정되어 있으므로, 회원 가입 시 이름은 반드시 입력해야 함
   - 만약 이름을 빼고 INSERT 시도 할 때?
```sql
-- `name` 열을 빼고 INSERT를 시도
INSERT INTO customers (email, password, address)
VALUES ('noname@example.com', 'password123', '서울시 마포구');
```
   - 실행 결과
```
Error Code: 1364. Field 'name' doesn't have a default value
```
   - 데이터베이스는 "name 필드에 기본값이 없다"라는 오류를 반환하며 INSERT를 거부
   - name열은 NOT NULL 인데 값을 주지 않았고, DEFAULT로 지정된 값도 없으므로 어떤 데이터를 넣어야 할지 알 수 없기 때문임
   - 이처럼 NOT NULL 제약 조건은 데이터의 완전성을 보장하는 첫 번째 방어선

5. UNIQUE 제약 조건 위반 : "이미 사용 중인 이메일입니다."
   - 가장 흔하게 마주치는 제약 조건 오류 중 하나는 UNIQUE 키 위반
   - customers 테이블의 email 열에는 UNIQUE 제약 조건이 걸려 있으므로, 모든 고객은 서로 다른 이메일 주소를 가져야 함
   - 만약 기존에 있는 이메일로 새로운 회원을 가입시키려고 하면 어떻게 될까?
```sql
INSERT INTO customers (name, email, password, address)
VALUES ('강감찬', 'kang@example.com', 'new_password_789', '서울시 강남구');
```
   - 강감찬 회원을 저장하는 쿼리는 문제 없이 실행되고, 데이터베이스에 저장
   - 실행 결과
```sql
SELECT * FROM customers;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/da57a690-b526-4176-85b5-7422c4318bbd">
</div>

```sql
-- 'kang@example.com'은 이미 '강감찬' 고객이 사용 중인 이메일
INSERT INTO customers (name, email, password, address)
VALUES ('홍길동', 'kang@example.com', 'new_password_123', '서울시 송파구');
```

   - 추가로 저장할 홍길동 회원의 이메일 ```kang@example.com```은 이미 강감찬 고객이 사용 중인 이메일
   - 실행 결과
```
Error Code: 1062. Duplicate entry 'kang@example.com' for key 'customers.email'
```
   - 홍길동 회원을 저장할 때 데이터베이스는 "customers.email 키에 중복된 값 ```kang@example.com```이 있습니다"라는 명확한 오류 메시지를 반환하며 INSERT 요청을 거부
   - 이처럼 UNIQUE 제약 조건은 데이터의 고유성을 보장하는 중요한 역할

6. 외래 키(FK) 제약 조건 : 관계의 무결성 지키기
   - 이제 관계형 데이터베이스의 핵심, 외래 키(Foreign Key) 제약 조건
   - orders 테이블은 customers 테이블과 products 테이블에 의존하는 '자식 테이블'
   - 즉, orders 테이블에 데이터를 추가하려면, 그 주문을 한 고객(customer_id)과 주문된 상품(product_id)이 반드시 customers 와 products 테이블에 실제로 존재해야 함
   - 먼저, 정상적인 주문 데이터를 입력
     + customers 테이블에 customer_id가 1인 고객이 있고, products 테이블에 product_id가 1인 '베이직 반팔 티셔츠'가 있다고 가정
     + 고객은 앞서 저장한 강감찬 고객(customer_id=1)이 있으니 상품만 하나 추가
```sql
INSERT INTO products (name, price, stock_quantity)
VALUES ('베이직 반팔 티셔츠', 19900, 200);
```
   - 실행 결과
```sql
SELECT * FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/65118efa-4e8c-479e-be17-ca5412bc15a5">
</div>

```sql
-- 1번 고객이 1번 상품을 1개 주문
INSERT INTO orders (customer_id, product_id, quantity)
VALUES (1, 1, 1);
```
   - 실행 결과
```sql
SELECT * FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/7e1f0d6c-aee2-45b8-a90a-1238db8bf53c">
</div>

   - 이 INSERT 문은 성공적으로 실행 : customer_id=1과 product_id=1이 부모 테이블에 모두 존재하기 때문임
   - order_id와 order_date, status 는 각각 AUTO_INCREMENT와 DEFAULT 설정에 따라 자동으로 채워졌음
   - 만약 실패한다면 customers 테이블에 customer_id가 없거나, products 테이블에 product_id가 존재하지 않을 것

7. 외래 키(FK) 제약 조건 위반 : "존재하지 않는 고객의 주문입니다."
   - 이번에는 존재하지 않는 고객의 주문을 넣어볼 것 : customers 테이블에는 customer_id가 999인 고객이 없음
```sql
-- 존재하지 않는 999번 고객이 1번 상품을 1개 주문하려고 시도한다.
INSERT INTO orders (customer_id, product_id, quantity)
VALUES (999, 1, 1);
```
   - 실행 결과
```
Error Code: 1452. Cannot add or update a child row: a foreign key constraint
fails (`my_shop`.`orders`, CONSTRAINT `fk_orders_customers` FOREIGN KEY
(`customer_id`) REFERENCES `customers` (`customer_id`))
```
   - 데이터베이스는 "자식 행을 추가하거나 업데이트할 수 없습니다 : fk_orders_customers 외래 키 제약 조건이 실패했습니다"라는 오류를 뱉어냄
   - 이 메시지는 orders 테이블에 INSERT 하려던 값 999 가 부모 테이블인 customers의 customer_id에 존재하지 않아서 외래 키 제약조건을 위반했다는 뜻
   - 이처럼 외래 키 제약 조건은 데이터의 정합성과 무결성을 지키는 핵심 장치
   - 이 기능 덕분에 우리는 유령 회원이나 단종된 상품이 주문되는 것과 같은 데이터 불일치 상황을 원천적으로 방지할 수 있음

8. 참고 : 외래 키 제약조건을 포함한 제약조건들은 INSERT 뿐만 아니라 UPDATE, DELETE 등 모든 상황의 데이터 불일치를 원천 방지
