-----
### 쇼핑몰 DDL과 DB 만들기
-----
1. 역정규화 전략과 감사 컬럼 추가를 모두 반영하여, 쇼핑몰 데이터베이스의 최종 DDL을 완성
   - member 테이블
```sql
CREATE TABLE member (
     member_id BIGINT NOT NULL AUTO_INCREMENT, -- 회원 ID (PK)
     login_id VARCHAR(50) NOT NULL, -- 로그인 ID
     password VARCHAR(255) NOT NULL, -- 비밀번호 (암호화)
     member_name VARCHAR(50) NOT NULL, -- 이름
     email VARCHAR(100) NOT NULL, -- 이메일
     addr VARCHAR(255) NULL, -- 주소

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (member_id),
     UNIQUE KEY uq_login_id (login_id),
     UNIQUE KEY uq_email (email)
);
```
   - product 테이블
```sql
CREATE TABLE product (
     product_id BIGINT NOT NULL AUTO_INCREMENT, -- 상품 ID (PK)
     product_name VARCHAR(100) NOT NULL, -- 상품명
     product_price INT NOT NULL, -- 가격
     stock_quantity INT NOT NULL DEFAULT 0, -- 재고 수량

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
     PRIMARY KEY (product_id),
     INDEX idx_product_name (product_name) -- 상품명 검색용 인덱스
);
```
   - idx_product_name 인덱스를 추가
   - orders 테이블 (역정규화, 인덱스 적용)
```sql
CREATE TABLE orders (
     order_id BIGINT NOT NULL AUTO_INCREMENT, -- 주문 ID (PK)
     member_id BIGINT NOT NULL, -- 회원 ID (FK)
     ordered_at DATETIME NOT NULL, -- 주문일 (애플리케이션에서 생성)
     order_status VARCHAR(20) NOT NULL DEFAULT 'ORDERED', -- 주문 상태
     total_amount INT NOT NULL, -- 총 주문 금액 (역정규화)

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (order_id),
     CONSTRAINT fk_orders_member FOREIGN KEY (member_id) REFERENCES member (member_id),
     INDEX idx_order_status_ordered_at (order_status, ordered_at) -- 관리자용 주문 조회 인덱스
);
```
   - total_amount 파생 컬럼(역정규화)이 추가 : 이를 통해 매번 총액을 계산하는 복잡한 집계 쿼리를 피할 수 있음
   - idx_order_status_ordered_at 복합 인덱스가 추가 :  관리자가 주문 상태와 기간으로 주문을 조회 할 때 성능을 향상시킴
   - order_item 테이블 (역정규화 적용)
```sql
CREATE TABLE order_item (
     order_item_id BIGINT NOT NULL AUTO_INCREMENT, -- 주문 상품 ID (PK)
     order_id BIGINT NOT NULL, -- 주문 ID (FK)
     product_id BIGINT NOT NULL, -- 상품 ID (FK)
     product_name VARCHAR(100) NOT NULL, -- 주문 당시 상품명 (역정규화)
     order_price INT NOT NULL, -- 주문 당시 가격
     order_quantity INT NOT NULL, -- 주문 수량

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (order_item_id),
     CONSTRAINT fk_order_item_orders FOREIGN KEY (order_id) REFERENCES orders (order_id),
     CONSTRAINT fk_order_item_product FOREIGN KEY (product_id) REFERENCES product (product_id)
);
```
   - product_name 컬럼이 추가 : 주문 내역 조회 시 product 테이블과의 JOIN 없이 상품명을 바로 확인 할 수 있음
   - delivery 테이블 (통합하지 않고 유지)
```sql
CREATE TABLE delivery (
     delivery_id BIGINT NOT NULL AUTO_INCREMENT, -- 배송 ID (PK)
     order_id BIGINT NOT NULL, -- 주문 ID (FK)
     delivery_status VARCHAR(20) NOT NULL DEFAULT 'READY',-- 배송 상태
     tracking_no VARCHAR(50) NULL, -- 운송장 번호
     ship_addr VARCHAR(255) NOT NULL, -- 배송지

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (delivery_id),
     UNIQUE KEY uq_delivery_order_id (order_id),
     CONSTRAINT fk_delivery_orders FOREIGN KEY (order_id) REFERENCES orders (order_id)
);
```

   - delivery 테이블은 orders와 통합하지 않고 독립적으로 유지 : 이를 통해 데이터 모델의 유연성과 확장성을 확보
   - pay 테이블 생성 DDL
```sql
CREATE TABLE pay (
 pay_id BIGINT NOT NULL AUTO_INCREMENT, -- 결제 ID (PK)
 order_id BIGINT NOT NULL, -- 주문 ID (FK, Unique)
 pay_method VARCHAR(50) NOT NULL, -- 결제 수단
 pay_amount INT NOT NULL, -- 결제 금액
 pay_status VARCHAR(20) NOT NULL, -- 결제 상태
 paid_at DATETIME NULL, -- 결제 완료일

 created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
 updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

 PRIMARY KEY (pay_id),
 UNIQUE KEY uq_pay_order_id (order_id), -- 주문 하나당 결제는 하나만 존재해야 함 (1:1 관계 보장)
 CONSTRAINT fk_pay_orders FOREIGN KEY (order_id) REFERENCES orders (order_id)
);
```

2. 이로써 우리는 정규화된 논리적 모델을 기반으로, 실제 운영 환경의 성능을 고려한 전략적인 역정규화, 그리고 데이터의 변경 이력을 추적하기 위한 감사 컬럼까지 모두 적용하여 최종 물리적 모델을 완성
3. 이제 이 DDL을 사용해 실제 데 이터베이스에 테이블을 생성하고 우리 쇼핑몰 서비스를 만들어나갈 준비를 마친 것
4. 쇼핑몰 DB 만들기
   - 이제 최종적으로 완성된 DDL과 샘플 데이터까지 포함하여,'쇼핑몰' 데이터베이스를 처음부터 끝까지 완전히 구축할 수 있는 통합 스크립트
   - 이 스크립트 하나만 있으면, 언제 어디서든 동일한 구조와 데이터를 가진 데이터베이스를 즉시 생성할 수 있음
   - 스크립트는 다음과 같은 순서로 구성
     + 데이터베이스 초기화 : 만약 기존에 my_shop3_ex 데이터베이스가 있다면 삭제하고, 깨끗한 상태에서 새로 생성
     + 테이블 초기화 : 생성할 테이블들이 이미 존재한다면 먼저 삭제하고, 외래 키(FK) 종속성을 고려하여 자식 테이블부터 삭제해야 함
     + 테이블 생성 (DDL) : member, product, orders, order_item, delivery, pay 순서로 테이블을 생성
     + 샘플 데이터 입력 (DML) : 생성된 테이블에 실제 운영 상황과 유사한 샘플 데이터를 입력하여, 이후 개발 및 테스트에 활용할 수 있도록 함

   - 쇼핑몰 DB 구축 통합 스크립트 - DDL
```sql
------------------------------------------
-- 데이터베이스 초기화
------------------------------------------
DROP DATABASE IF EXISTS my_shop3_ex;
CREATE DATABASE my_shop3_ex;
USE my_shop3_ex;

------------------------------------------
-- 테이블 초기화 (외래 키 종속성을 고려하여 자식 테이블부터 삭제)
------------------------------------------
DROP TABLE IF EXISTS pay;
DROP TABLE IF EXISTS delivery;
DROP TABLE IF EXISTS order_item;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS product;
DROP TABLE IF EXISTS member;

-- 1. member 테이블
CREATE TABLE member (
     member_id BIGINT NOT NULL AUTO_INCREMENT, -- 회원 ID (PK)
     login_id VARCHAR(50) NOT NULL, -- 로그인 ID
     password VARCHAR(255) NOT NULL, -- 비밀번호 (암호화)
     member_name VARCHAR(50) NOT NULL, -- 이름
     email VARCHAR(100) NOT NULL, -- 이메일
     addr VARCHAR(255) NULL, -- 주소
     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (member_id),
     UNIQUE KEY uq_login_id (login_id),
     UNIQUE KEY uq_email (email)
);

-- 2. product 테이블
CREATE TABLE product (
     product_id BIGINT NOT NULL AUTO_INCREMENT, -- 상품 ID (PK)
     product_name VARCHAR(100) NOT NULL, -- 상품명
     product_price INT NOT NULL, -- 가격
     stock_quantity INT NOT NULL DEFAULT 0, -- 재고 수량
    
     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
     PRIMARY KEY (product_id),
     INDEX idx_product_name (product_name) -- 상품명 검색용 인덱스
);

-- 3. orders 테이블
CREATE TABLE orders (
     order_id BIGINT NOT NULL AUTO_INCREMENT, -- 주문 ID (PK)
     member_id BIGINT NOT NULL, -- 회원 ID (FK)
     ordered_at DATETIME NOT NULL, -- 주문일 (애플리케이션에서 생성)
     order_status VARCHAR(20) NOT NULL DEFAULT 'ORDERED', -- 주문 상태
     total_amount INT NOT NULL, -- 총 주문 금액 (역정규화)

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (order_id),
     CONSTRAINT fk_orders_member FOREIGN KEY (member_id) REFERENCES member (member_id),
     INDEX idx_order_status_ordered_at (order_status, ordered_at) -- 관리자용 주문 조회 인덱스
);

-- 4. order_item 테이블
CREATE TABLE order_item (
     order_item_id BIGINT NOT NULL AUTO_INCREMENT, -- 주문 상품 ID (PK)
     order_id BIGINT NOT NULL, -- 주문 ID (FK)
     product_id BIGINT NOT NULL, -- 상품 ID (FK)
     product_name VARCHAR(100) NOT NULL, -- 주문 당시 상품명 (역정규화)
     order_price INT NOT NULL, -- 주문 당시 가격
     order_quantity INT NOT NULL, -- 주문 수량

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (order_item_id),
     CONSTRAINT fk_order_item_orders FOREIGN KEY (order_id) REFERENCES orders (order_id),
     CONSTRAINT fk_order_item_product FOREIGN KEY (product_id) REFERENCES product (product_id)
);

-- 5. delivery 테이블
CREATE TABLE delivery (
     delivery_id BIGINT NTO NULL AUTO_INCREMENT, -- 배송 ID (PK)
     order_id BIGINT NOT NULL, -- 주문 ID (FK, Unique)
     delivery_status VARCHAR(20) NOT NULL DEFAULT 'READY',-- 배송 상태
     tracking_no VARCHAR(50) NULL, -- 운송장 번호
     ship_addr VARCHAR(255) NOT NULL, -- 배송지

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (delivery_id),
     UNIQUE KEY uq_delivery_order_id (order_id), -- 주문 하나당 배송은 하나
     CONSTRAINT fk_delivery_orders FOREIGN KEY (order_id) REFERENCES orders (order_id)
);

-- 6. pay 테이블
CREATE TABLE pay (
     pay_id BIGINT NOT NULL AUTO_INCREMENT, -- 결제 ID (PK)
     order_id BIGINT NOT NULL, -- 주문 ID (FK, Unique)
     pay_method VARCHAR(50) NOT NULL, -- 결제 수단
     pay_amount INT NOT NULL, -- 결제 금액
     pay_status VARCHAR(20) NOT NULL, -- 결제 상태
     paid_at DATETIME NULL, -- 결제 완료일

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (pay_id),
     UNIQUE KEY uq_pay_order_id (order_id), -- 주문 하나당 결제는 하나
     CONSTRAINT fk_pay_orders FOREIGN KEY (order_id) REFERENCES orders (order_id)
);
```

   - 쇼핑몰 DB 구축 통합 스크립트 - 샘플 DML
```sql
------------------------------------------
-- 샘플 데이터 입력 (DML)
------------------------------------------
-- 1. 회원 데이터
-- 실제 서비스에서는 비밀번호를 반드시 암호화(해싱)하여 저장해야 함
INSERT INTO member(login_id, password, member_name, email, addr)
VALUES
('sejong', 'pass123!', '세종대왕', 'sejong@example.com', '서울시 종로구'),
('sunsin', 'pass456!', '이순신', 'sunsin@example.com', '전라남도 여수시'),
('curie', 'pass789!', '마리 퀴리', 'curie@example.com', '프랑스 파리'),
('nate', 'pass101!', '네이트', 'nate@example.com', '서울시 송파구'),
('jobs', 'pass112!', '스티브 잡스', 'jobs@example.com', '미국 캘리포니아'),
('sean', 'pass131!', '션', 'sean@example.com', '성남시 분당구');

-- 2. 상품 데이터 (IT 기기 및 전문 서적)
INSERT INTO product(product_name, product_price, stock_quantity)
VALUES
('싱크패드 노트북', 3500000, 15),
('리얼포스 키보드', 380000, 40),
('델 모니터', 1200000, 25),
('JPA 프로그래밍 책', 32000, 150),
('로지텍 지슈라2 마우스', 139000, 200),
('로지텍 웹캠', 450000, 30),
('갤럭시 S25 휴대폰', 1800000, 50),
('아이폰 17', 1900000, 50);

-- 3. 주문 데이터 & 관련 데이터 (주문, 주문상품, 배송, 결제)
-- 시나리오 1 : 세종대왕, 'JPA 프로그래밍 책' 도서 1권 주문 (배송 완료)
INSERT INTO orders(member_id, ordered_at, order_status, total_amount)
VALUES
(1, '2025-09-05 10:00:00', 'COMPLETED', 32000);

SET @last_order_id = LAST_INSERT_ID();

INSERT INTO order_item(order_id, product_id, product_name, order_price, order_quantity)
VALUES
(@last_order_id, 4, 'JPA 프로그래밍 책', 32000, 1);

INSERT INTO delivery(order_id, delivery_status, tracking_no, ship_addr)
VALUES
(@last_order_id, 'COMPLETED', 'KR13970515', '서울시 종로구 경복궁');

INSERT INTO pay(order_id, pay_method, pay_amount, pay_status, paid_at)
VALUES
(@last_order_id, 'BANK_TRANSFER', 32000, 'PAID', '2025-09-05 10:05:00');

-- 시나리오 2 : 이순신, 전투 지휘용 '델 모니터' 1대 주문 (배송중)
INSERT INTO orders(member_id, ordered_at, order_status, total_amount)
VALUES
(2, '2025-09-15 14:30:00', 'SHIPPING', 1200000);

SET @last_order_id = LAST_INSERT_ID();

INSERT INTO order_item(order_id, product_id, product_name, order_price,
order_quantity)
VALUES
(@last_order_id, 3, '델 모니터', 1200000, 1);

INSERT INTO delivery(order_id, delivery_status, tracking_no, ship_addr)
VALUES
(@last_order_id, 'SHIPPING', 'KR15450428', '전라남도 여수시 진남관');

INSERT INTO pay(order_id, pay_method, pay_amount, pay_status, paid_at)
VALUES
(@last_order_id, 'CARD', 1200000, 'PAID', '2025-09-15 14:32:00');

-- 시나리오 3 : 네이트, '싱크패드 노트북'과 '리얼포스 키보드' 주문 (배송 준비)
INSERT INTO orders(member_id, ordered_at, order_status, total_amount)
VALUES
(4, '2025-09-20 09:00:00', 'ORDERED', 3880000);

SET @last_order_id = LAST_INSERT_ID();

INSERT INTO order_item(order_id, product_id, product_name, order_price,
order_quantity)
VALUES
(@last_order_id, 1, '싱크패드 노트북', 3500000, 1),
(@last_order_id, 2, '리얼포스 키보드', 380000, 1);

INSERT INTO delivery(order_id, delivery_status, ship_addr)
VALUES
(@last_order_id, 'READY', '서울시 송파구 잠실동');

INSERT INTO pay(order_id, pay_method, pay_amount, pay_status, paid_at)
VALUES
(@last_order_id, 'CARD', 3880000, 'PAID', '2025-09-20 09:01:00');

-- 시나리오 4 : 스티브 잡스, '갤럭시 S25 휴대폰'을 주문했으나 취소
INSERT INTO orders(member_id, ordered_at, order_status, total_amount)
VALUES
(5, '2025-09-11 18:00:00', 'CANCELED', 1800000);

SET @last_order_id = LAST_INSERT_ID();

INSERT INTO order_item(order_id, product_id, product_name, order_price,
order_quantity)
VALUES
(@last_order_id, 7, '갤럭시 S25 휴대폰', 1800000, 1);

INSERT INTO delivery(order_id, delivery_status, tracking_no, ship_addr)
VALUES
(@last_order_id, 'SHIPPING', 'EA12341234', '미국 캘리포니아');

INSERT INTO pay(order_id, pay_method, pay_amount, pay_status)
VALUES
(@last_order_id, 'CARD', 1800000, 'CANCELED');

-- 시나리오 5 : 션, 'JPA 프로그래밍 책'과 '로지텍 지슈라2 마우스' 주문 (배송 완료)
INSERT INTO orders(member_id, ordered_at, order_status, total_amount)
VALUES
(6, '2025-08-25 21:00:00', 'COMPLETED', 171000);

SET @last_order_id = LAST_INSERT_ID();

INSERT INTO order_item(order_id, product_id, product_name, order_price,
order_quantity)
VALUES
(@last_order_id, 4, 'JPA 프로그래밍 책', 32000, 1),
(@last_order_id, 5, '로지텍 지슈라2 마우스', 139000, 1);

INSERT INTO delivery(order_id, delivery_status, tracking_no, ship_addr)
VALUES
(@last_order_id, 'COMPLETED', 'KR19691228', '성남시 분당구 판교동');

INSERT INTO pay(order_id, pay_method, pay_amount, pay_status, paid_at)
VALUES
(@last_order_id, 'CARD', 171000, 'PAID', '2025-08-25 21:05:00');

-- 시나리오 6 : 션, '싱크패드 노트북'과 '로지텍 웹캠' 추가 주문 (배송중)
INSERT INTO orders(member_id, ordered_at, order_status, total_amount)
VALUES
(6, '2025-09-22 13:10:00', 'SHIPPING', 3950000);

SET @last_order_id = LAST_INSERT_ID();

INSERT INTO order_item(order_id, product_id, product_name, order_price,
order_quantity)
VALUES
(@last_order_id, 1, '싱크패드 노트북', 3500000, 1),
(@last_order_id, 6, '로지텍 웹캠', 450000, 1);

INSERT INTO delivery(order_id, delivery_status, tracking_no, ship_addr)
VALUES
(@last_order_id, 'SHIPPING', 'KR20250922', '성남시 분당구 정자동');

INSERT INTO pay(order_id, pay_method, pay_amount, pay_status, paid_at)
VALUES
(@last_order_id, 'CARD', 3950000, 'PAID', '2025-09-22 13:11:00');
```
