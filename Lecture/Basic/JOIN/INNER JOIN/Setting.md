-----
### 실습 데이터 준비
-----
1. 주요 비즈니스 규칙 및 제약사항
   - 고객 가입 : 모든 고객은 고유한 이메일 주소를 가져야 함 (이름과 이메일은 필수 정보)
   - 주문 생성 : 주문은 반드시 특정 고객(user_id)과 특정 상품(product_id)에 연결되어야 함
     + 하나의 주문에 한 종류의 상품만 선택할 수 있고, 상품의 수량은 선택할 수 있음
   - 주문 상태 관리 : 주문이 생성되면 기본 상태는 'PENDING'이며, 이후 'COMPLETED', 'SHIPPED', 'CANCELLED'로 변경될 수 있음
     + PENDING(대기)
     + SHIPPED(배송)
     + COMPLETED(완료)
     + CANCELLED(취소)
   - 재고 관리 : 주문이 발생하면 해당 products 테이블의 stock_quantity(재고)는 주문 quantity(수량)만큼 차감되어야 함 (이 로직은 데이터베이스가 아니라 애플리케이션에서 구현해야 함)
   - 직원 관리 구조 : 직원은 매니저를 가질 수 있으며, 매니저 또한 직원으로, 매니저가 없는 최상위 직원이 존재할 수 있음

2. 왜 이런 비즈니스 규칙과 제약사항을 먼저 살펴보는 이유
   - 실제로 데이터베이스를 만들 때는 먼저 어떤 데이터가 필요하고, 그 데이터들이 어떻게 연결되는지 설계하는 과정이 꼭 필요

3. 데이터 모델 다이어그램 (ERD)
<div align="center">
<img src="https://github.com/user-attachments/assets/4f944376-b744-440e-ab11-5ecdf6043d58">
</div>

   - 고객 주문 상품의 관계
     + 고객(users)은 여러 개의 주문(orders)을 생성할 수 있음
     + 상품(products)은 여러 주문(orders)에 포함될 수 있음
     + 하나의 주문(orders)은 한 명의 고객과 하나의 상품에 연결

   - 나머지 관계
     + 직원(employees)은 다른 직원을 관리하는 계층 구조(SELF JOIN)를 가짐
     + manager_id를 통해 상사를 알 수 있음
     + 사이즈(sizes)와 색상(colors) 테이블은 상품의 모든 옵션 조합을 생성하기 위해 사용

4. 테이블별 상세 요구사항
   - 고객(users) 테이블 : 고객의 개인 정보 및 계정 정보를 저장
<div align="center">
<img src="https://github.com/user-attachments/assets/a8bca084-c9da-4a1a-bd29-e77505ba76ac">
</div>

   - 상품 (products) 테이블 : 판매하는 상품의 정보를 관리
<div align="center">
<img src="https://github.com/user-attachments/assets/a181a5fc-5fe7-47c8-8fee-036a42d8cc27">
</div>

   - 주문(orders) 테이블 : 고객의 상품 주문 내역을 기록
<div align="center">
<img src="https://github.com/user-attachments/assets/68da1d5e-10b2-40f9-b2df-eb0f0f671a21">
</div>

   - 직원(employees) 테이블 : 직원 및 관리자(매니저) 관계를 정의 (Self Join 관계)
<div align="center">
<img src="https://github.com/user-attachments/assets/6c19469a-5cef-44e9-9468-3e6ba46beb98">
</div>

  - 사이즈(sizes) 및 색상(colors) 테이블 : 상품의 다양한 옵션을 조합하기 위한 기준 데이터를 정의 (CROSS JOIN 실습용)
<div align="center">
<img src="https://github.com/user-attachments/assets/e5a21c80-fddb-4df1-93d1-a6d702444953">
</div>

5. 테이블 설계 및 생성
   - 데이터베이스 이름의 경우 my_shop2로 진행
```sql
-- 데이터베이스가 존재하지 않으면 생성
CREATE DATABASE IF NOT EXISTS my_shop2;
USE my_shop2;

-- 테이블이 존재하면 삭제 (실습을 위해 초기화)
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS users;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS employees;
DROP TABLE IF EXISTS sizes;
DROP TABLE IF EXISTS colors;

-- 고객 테이블 생성
CREATE TABLE users (
     user_id BIGINT AUTO_INCREMENT,
     name VARCHAR(255) NOT NULL,
     email VARCHAR(255) NOT NULL UNIQUE,
     address VARCHAR(255),
     birth_date DATE,
     created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
     PRIMARY KEY (user_id)
);

-- 상품 테이블 생성
CREATE TABLE products (
     product_id BIGINT AUTO_INCREMENT,
     name VARCHAR(255) NOT NULL,
     category VARCHAR(100),
     price INT NOT NULL,
     stock_quantity INT NOT NULL,
     PRIMARY KEY (product_id)
);

-- 주문 테이블 생성
CREATE TABLE orders (
     order_id BIGINT AUTO_INCREMENT,
     user_id BIGINT NOT NULL,
     product_id BIGINT NOT NULL,
     order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
     quantity INT NOT NULL,
     status VARCHAR(50) DEFAULT 'PENDING',
     PRIMARY KEY (order_id),
     CONSTRAINT fk_orders_users FOREIGN KEY (user_id)
     REFERENCES users(user_id),
     CONSTRAINT fk_orders_products FOREIGN KEY (product_id)
     REFERENCES products(product_id)
);

-- 직원 테이블 생성 (SELF JOIN 실습용)
CREATE TABLE employees (
     employee_id BIGINT AUTO_INCREMENT,
     name VARCHAR(255) NOT NULL,
     manager_id BIGINT,
     PRIMARY KEY (employee_id),
     FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

-- 사이즈 테이블 (CROSS JOIN 실습용)
CREATE TABLE sizes (
     size VARCHAR(10) PRIMARY KEY
);

-- 색상 테이블 (CROSS JOIN 실습용)
CREATE TABLE colors (
     color VARCHAR(20) PRIMARY KEY
);
```
   - CREATE DATABASE IF NOT EXISTS my_shop2 : my_shop2 데이터베이스가 존재하지 않으면 생성
   - DROP TABLE IF EXISTS orders : 주문 테이블이 만약에 존재하면 DROP
   - 이렇게 IF EXISTS 구문을 활용하면 기존에 테이블이 있는 경우에만 깔끔하게 제거할 수 있음
     + 덕분에 같은 구문을 여러 번 실행해도 오류가 발생하지 않음
     + 만약 테이블이 없다면 DROP을 할 수 없기 때문에 DROP TABLE에서 오류가 발생
     + IF EXISTS 구문은 이런 번거로움을 해결

6. 샘플 데이터 입력
   - 한 번도 주문하지 않은 고객(레오나르도 다빈치)과 한 번도 팔리지 않은 상품(고급 가죽 지갑)을 포함
```sql
-- 고객 데이터 입력
INSERT INTO users(name, email, address, birth_date) VALUES
('션', 'sean@example.com', '서울시 강남구', '1990-01-15'),
('네이트', 'nate@example.com', '경기도 성남시', '1988-05-22'),
('세종대왕', 'sejong@example.com', '서울시 종로구', '1397-05-15'),
('이순신', 'sunsin@example.com', '전라남도 여수시', '1545-04-28'),
('마리 퀴리', 'marie@example.com', '서울시 강남구', '1867-11-07'),
('레오나르도 다빈치', 'vinci@example.com', '이탈리아 피렌체', '1452-04-15');

-- 상품 데이터 입력
INSERT INTO products(name, category, price, stock_quantity) VALUES
('프리미엄 게이밍 마우스', '전자기기', 75000, 50),
('기계식 키보드', '전자기기', 120000, 30),
('4K UHD 모니터', '전자기기', 350000, 20),
('관계형 데이터베이스 입문', '도서', 28000, 100),
('고급 가죽 지갑', '패션', 150000, 15),
('스마트 워치', '전자기기', 280000, 40);

-- 주문 데이터 입력
INSERT INTO orders(user_id, product_id, quantity, status, order_date) VALUES
(1, 1, 1, 'COMPLETED', '2025-06-10 10:00:00'),
(1, 4, 2, 'COMPLETED', '2025-06-10 10:05:00'),
(2, 2, 1, 'SHIPPED', '2025-06-11 14:20:00'),
(3, 4, 1, 'COMPLETED', '2025-06-12 09:00:00'),
(4, 3, 1, 'PENDING', '2025-06-15 11:30:00'),
(5, 1, 1, 'COMPLETED', '2025-06-16 18:00:00'),
(2, 1, 2, 'SHIPPED', '2025-06-17 12:00:00');

-- 직원 데이터 입력
INSERT INTO employees(employee_id, name, manager_id) VALUES
(1, '김회장', NULL),
(2, '박사장', 1),
(3, '이부장', 2),
(4, '최과장', 3),
(5, '정대리', 4),
(6, '홍사원', 4);

-- 사이즈 데이터 입력
INSERT INTO sizes(size) VALUES
('S'), ('M'), ('L'), ('XL');

-- 색상 데이터 입력
INSERT INTO colors(color) VALUES
('Red'), ('Blue'), ('Black')
```

7. 준비된 데이터 확인
    - users 테이블
```sql
SELECT *
FROM users;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/bdb771b1-78d0-49a0-84ad-261e9b089bbb">
</div>

   - created_at 필드(컬럼)의 날짜는 생성한 날짜이므로 각각 다름

   - products 테이블
```sql
SELECT *
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a897948c-9ddf-42ed-8a02-80d30b8285b9">
</div>

   - orders 테이블
```sql
SELECT *
FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/fcffae49-a1bb-4cc2-a190-28780db49454">
</div>
