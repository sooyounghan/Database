-----
### 문제와 풀이
-----
1. 문제 1 : 데이터베이스와 테이블 생성하기
   - 쇼핑몰 프로젝트를 위해 my_test 데이터베이스를 생성하고, 해당 데이터베이스를 사용하도록 선택
   - 그 후, 고객 정보를 저장할 members 테이블을 생성하며, members 테이블은 다음과 같은 열(column)을 가짐
     + id : 정수(INT), 기본키 설정
     + name : 최대 50글자의 문자열(VARCHAR), NOT NULL 제약 조건
     + join_date : 날짜(DATE)
   - DESC members; 실행 시 다음과 같은 결과가 출력
<div align="center">
<img src="https://github.com/user-attachments/assets/058c2964-7fb9-4120-abc7-5ac4df0ec619">
</div>

   - 정답
```sql
-- 1. my_test 데이터베이스 생성
CREATE DATABASE my_test;

-- 2. my_test 데이터베이스 선택
USE my_test;

-- 3. members 테이블 생성
CREATE TABLE members (
     id INT PRIMARY KEY, -- 기본키 설정
     name VARCHAR(50) NOT NULL,
     join_date DATE
);

-- 4. 테이블 구조 확인
DESC members;
```

2. 문제2 : 데이터 추가 및 조회하기 (INSERT, SELECT)
   - 1번 문제에서 생성한 members 테이블에 아래 두 명의 회원 데이터를 추가하고, 테이블의 전체 내용을 조회
      + ID : 1, 이름 : 션, 가입일 : 2025-01-10
      + ID : 2, 이름 : 네이트, 가입일 : 2025-02-15
   - SELECT * FROM members; 실행 시 다음과 같은 결과가 출력
<div align="center">
<img src="https://github.com/user-attachments/assets/370997af-89c4-4eec-96be-10f4a27123e9">
</div>

   - 정답
```sql
-- 1. 첫 번째 회원 데이터 추가
INSERT INTO members (id, name, join_date)
VALUES (1, '션', '2025-01-10');

-- 2. 두 번째 회원 데이터 추가
INSERT INTO members (id, name, join_date)
VALUES (2, '네이트', '2025-02-15');

-- 3. 전체 데이터 조회
SELECT * FROM members;
```

3. 문제 3 : 데이터 수정 및 삭제하기 (UPDATE, DELETE)
   - 2번 문제에서 추가한 데이터에 다음 두 가지 작업을 수행하고, 최종 결과를 조회
      + id가 2인 회원 '네이트'의 이름을 '네이트2'로 변경
      + id가 1인 회원 '션'의 정보를 삭제
   - SELECT * FROM members; 실행 시 다음과 같은 결과가 출력
<div align="center">
<img src="https://github.com/user-attachments/assets/bc8e7d18-86b0-474c-ae01-a0ce6a159859">
</div>

   - 정답
```sql
-- 1. 이름 변경 (UPDATE)
UPDATE members
SET name = '네이트2'
WHERE id = 2;

-- 2. 회원 정보 삭제 (DELETE)
DELETE FROM members
WHERE id = 1;

-- 3. 최종 데이터 조회
SELECT * FROM members;
```

4. 문제 4 : 제약 조건을 포함한 테이블 생성하기
  - 쇼핑몰의 products (상품) 테이블을 다음 요구사항과 제약 조건에 맞게 생성하고, 테이블 구조를 확인 (데이터베이스가 my_test 로 다르기 때문에 기존 테이블에 영향을 주지 않음) 
    + product_id : 정수, 자동으로 1씩 증가하는 기본 키(PRIMARY KEY)
    + product_name : 최대 100글자의 문자열, 비어 있을 수 없음(NOT NULL)
    + product_code : 최대 20글자의 문자열, 값이 중복될 수 없음(UNIQUE)
    + price : 정수, 비어 있을 수 없음(NOT NULL)
    + stock_count : 정수, 비어 있을 수 없으며(NOT NULL), 값을 지정하지 않으면 기본으로 0이 입력됨 (DEFAULT)
  - DESC products; 실행 시 다음과 같은 결과가 출력
<div align="center">
<img src="https://github.com/user-attachments/assets/cd317580-d1ea-49e4-805c-9d7523057024">
</div>

  - 정답
```sql
CREATE TABLE products (
     product_id INT AUTO_INCREMENT PRIMARY KEY,
     product_name VARCHAR(100) NOT NULL,
     product_code VARCHAR(20) UNIQUE,
     price INT NOT NULL,
     stock_count INT NOT NULL DEFAULT 0
);
DESC products;
```

5. 문제 5: 외래 키(Foreign Key)로 테이블 관계 맺기
   - 고객(customers)과 주문(orders) 테이블을 생성
   - orders 테이블의 customer_id는 customers 테이블의 customer_id를 참조하는 외래 키(Foreign Key) 관계를 맺어야 함
   - customers 테이블
     + customer_id : 정수, 기본 키, 자동 증가
     + name : 문자열(50자), 필수

   - orders 테이블
     + order_id : 정수, 자동 증가 기본 키
     + customer_id : 정수, 필수
     + order_date : 날짜와 시간(DATETIME), 기본값은 현재 시각

   - FOREIGN KEY 설정 : orders.customer_id가 customers.customer_id를 참조
   - 두 테이블을 생성한 후, '홍길동' 고객과 그 고객이 주문한 데이터를 각각 1개씩 추가하고, 두 테이블의 전체 내용을 조회
```sql
SELECT * FROM customers;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/55742b15-f3cf-41c8-ae81-4efd35d0fc59">
</div>

```sql
SELECT * FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/27f21df8-f00f-47fd-8f5c-46cacd9253a0">
</div>

   - 정답
```sql
-- customers 테이블 생성
CREATE TABLE customers (
     customer_id INT AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(50) NOT NULL
);

-- orders 테이블 생성
CREATE TABLE orders (
     order_id INT AUTO_INCREMENT PRIMARY KEY,
     customer_id INT NOT NULL,
     order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
     CONSTRAINT fk_orders_customers FOREIGN KEY (customer_id)
     REFERENCES customers(customer_id)
);

-- 고객 데이터 추가
INSERT INTO customers (name) VALUES ('홍길동');

-- 주문 데이터 추가 (방금 추가된 홍길동 고객의 customer_id는 1)
INSERT INTO orders (customer_id) VALUES (1);

-- 데이터 조회
SELECT * FROM customers;
SELECT * FROM orders;
```

6. 문제 6 : 제약 조건 위반 상황 만들기
   - 5번 문제에서 생성한 테이블들을 이용해 아래 두 가지 잘못된 데이터 추가를 시도
   - customers 테이블에 존재하지 않는 customer_id(예) 999)를 사용하여 orders 테이블에 새로운 주문을 추가
   - customers 테이블에 고객을 추가할 때, 필수 항목인 name을 빼고 추가
```sql
-- 시도 1: 존재하지 않는 고객의 주문 추가
INSERT INTO orders (customer_id) VALUES (999);

-- 시도 2: 필수 항목(name) 누락
INSERT INTO customers (customer_id) VALUES (2)
```

  - 실행 결과 : 시도 1 결과
```
Error Code: 1452. Cannot add or update a child row: a foreign key constraint
fails (`my_test`.`orders`, CONSTRAINT `fk_orders_customers` FOREIGN KEY
(`customer_id`) REFERENCES `customers` (`customer_id`))
```
  - 실패 이유 : 외래 키(Foreign Key) 제약 조건 위반
    + orders 테이블에 주문을 추가하려면, customer_id에 해당하는 값이 부모 테이블인 customers에 반드시 존재해야 함
    + customer_id 999는 customers 테이블에 없으므로 데이터 추가가 거부

  - 실행 결과 : 시도 2 결과
```
Error Code: 1364. Field 'name' doesn't have a default value
```
  - 실패 이유 : NOT NULL 제약 조건 위반
    + customers 테이블의 name 열은 NOT NULL로 설정되어 있어 값을 반드시 입력해야 함
    + name 값을 지정하지 않고 추가를 시도했기 때문에 데이터 추가가 거부

7. 문제와 풀이가 끝나면 다음 명령을 실행해서 사용 데이터베이스를 원래대로 변경
```sql
USE my_shop;
```
