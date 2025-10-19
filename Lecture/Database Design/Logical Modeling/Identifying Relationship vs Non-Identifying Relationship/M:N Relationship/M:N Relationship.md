-----
### 식별 관계 vs 비식별 관계 - 다대다(M:N)
-----
1. 주문(orders)과 상품(product)의 다대다 관계를 해결하는 order_item 테이블 예시
   - 비즈니스 규칙
      + '하나의 주문에는 동일한 상품이 중복으로 들어가면 안 된다'는 아주 일반적인 비즈니스 규칙이 있다고 생각
      + 예를 들어, '주문번호 100번(order_id)'에 '청바지(product_id)' 상품이 두 번 기록되면 안 돰
      + 데이터가 중복으로 쌓이면 나중에 통계를 낼 때나 재고를 계산할 때 심각한 오류를 유발할 수 있음

2. 식별 관계 (Identifying) : 연결 테이블이 양쪽 부모 테이블의 PK를 모두 물려받아 복합키로 구성하는 방식
<div align="center">
<img src="https://github.com/user-attachments/assets/c2c628e2-15c4-455e-bd0c-dd1ee727c00d">
</div>

```sql
DROP TABLE IF EXISTS order_item_identifying;
DROP TABLE IF EXISTS product_identifying;
DROP TABLE IF EXISTS orders_identifying;

-- 상품 테이블 생성
CREATE TABLE product_identifying (
     product_id BIGINT NOT NULL, -- 상품id 직접 입력
     name VARCHAR(100) NOT NULL,
     price INT NOT NULL,
     PRIMARY KEY (product_id)
);

-- 주문 테이블 생성
CREATE TABLE orders_identifying (
     order_id BIGINT NOT NULL, -- 주문id 직접 입력
     order_date DATE,
     PRIMARY KEY (order_id)
);

CREATE TABLE order_item_identifying (
     order_id BIGINT NOT NULL, -- PK의 일부 + FK
     product_id BIGINT NOT NULL, -- PK의 일부 + FK
     order_price INT NOT NULL,
     order_quantity INT NOT NULL,

     PRIMARY KEY (order_id, product_id), -- 복합 기본 키
     CONSTRAINT fk_oi_iden_orders FOREIGN KEY (order_id)
     REFERENCES orders_identifying (order_id),
     CONSTRAINT fk_oi_iden_product FOREIGN KEY (product_id)
     REFERENCES product_identifying (product_id)
);
```
   - order_item_identifying 테이블의 PK는 (order_id, product_id) 복합키
   - 이 구조는 데이터베이스 레벨에서 '하나의 주문에 동일한 상품은 한 번만 포함될 수 있다'는 비즈니스 규칙을 PK를 통해 강제
   - 장점
     + 비즈니스 규칙 강제 : 복합 키(PK) 자체가 중복을 막아주므로 데이터 정합성을 확실히 보장
     + 인덱스 효율 : order_id와 product_id를 함께 조회하는 쿼리에서 복합키 인덱스가 효율적으로 동작할 수 있음
   - 단점
     + 유연성 부족 : 만약 '하나의 주문에 옵션이 다른 동일 상품을 여러 개 담을 수 있다'는 요구사항이 생기면 이 모델은 즉시 한계에 부딪힘
       * PK에 option_id 같은 컬럼을 또 추가해야만 하는데, PK는 불변해야 함
       * 이것은 PK를 바꾸는 매우 심각한 변경
     + 복잡성 증가 : 복합키는 관리 및 사용이 불편하고, ORM 매핑도 더 복잡해진짐

3. 비즈니스 규칙 만족
   - 식별 관계의 가장 큰 특징은 기본 키(PK) 자체가 비즈니스 규칙을 강제한다는 점
   - PRIMARY KEY (order_id, product_id) 설정은 order_id와 product_id의 조합이 테이블 전체에서 유일해야 함을 의미
   - 정상 데이터 추가 : 먼저 100번 주문에 1번 상품(청바지)과 2번 상품(티셔츠)을 하나씩 추가
```sql
-- 상품 데이터 삽입
INSERT INTO product_identifying (product_id, name, price)
VALUES (1, '청바지', 50000);
INSERT INTO product_identifying (product_id, name, price)
VALUES (2, '티셔츠', 25000);

-- 주문 100 생성
INSERT INTO orders_identifying (order_id, order_date)
VALUES(100, '2025-09-01');

-- 100번 주문에 1번 상품(청바지) 추가
INSERT INTO order_item_identifying (order_id, product_id, order_price, order_quantity)
VALUES (100, 1, 30000, 1);

-- 100번 주문에 2번 상품(티셔츠) 추가
INSERT INTO order_item_identifying (order_id, product_id, order_price, order_quantity)
VALUES (100, 2, 15000, 2);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/0da9e163-3a0b-4b97-bd97-f993026a88ab">
</div>

4. 중복 데이터 추가 시도
   - 이제 100번 주문에 1번 상품(청바지)을 한 번 더 추가하려고 시도
```sql
-- 100번 주문에 1번 상품(청바지)을 중복 추가 시도
INSERT INTO order_item_identifying (order_id, product_id, order_price, order_quantity)
VALUES (100, 1, 30000, 5); -- 수량을 5개로 다시 주문 시도
```
   - 실행 결과 : 데이터베이스는 이 요청을 거부하고 오류를 발생시킴
```
Error Code: 1062. Duplicate entry '100-1' for key
'order_item_identifying.PRIMARY'
```
   - 이 오류 메시지는 order_item_identifying 테이블의 기본 키에 100-1 이라는 값이 이미 존재하여 중복된다는 의미
   - 이처럼 식별 관계 모델은 애플리케이션 로직에 실수가 있더라도 데이터베이스의 PK 제약조건이 최후의 보루가 되어 데이터의 중복을 원천적으로 차단 : 요구사항을 매우 강력하고 확실하게 만족시키는 방법

5. 비식별 관계 (Non-identifying)
   - 쇼핑몰에 설계하고 사용했던 방식 : 연결 테이블이 자신만의 고유한 PK를 가짐
   - 테이블 설계
```sql
DROP TABLE IF EXISTS order_item_non_identifying;
DROP TABLE IF EXISTS product_non_identifying;
DROP TABLE IF EXISTS orders_non_identifying;

-- 상품 테이블 생성
CREATE TABLE product_non_identifying (
   product_id BIGINT NOT NULL, -- 상품id 직접 입력
   name VARCHAR(100) NOT NULL,
   price INT NOT NULL,
  
   PRIMARY KEY (product_id)
);

-- 주문 테이블 생성
CREATE TABLE orders_non_identifying (
   order_id BIGINT NOT NULL, -- 주문id 직접 입력
   order_date DATE,

   PRIMARY KEY (order_id)
);

CREATE TABLE order_item_non_identifying (
 order_item_id BIGINT NOT NULL AUTO_INCREMENT, -- 독립적인 PK
 order_id BIGINT NOT NULL, -- 일반 컬럼 + FK
 product_id BIGINT NOT NULL, -- 일반 컬럼 + FK
 order_price INT NOT NULL,
 order_quantity INT NOT NULL,

 PRIMARY KEY (order_item_id),
 CONSTRAINT fk_oi_non_orders FOREIGN KEY (order_id)
 REFERENCES orders_non_identifying(order_id),
 CONSTRAINT fk_oi_non_product FOREIGN KEY (product_id)
 REFERENCES product_non_identifying(product_id)
);
```
   - order_item_non_identifying 테이블은 비즈니스 의미가 없는 대리 키 order_item_id를 PK로 사용
   - order_id와 product_id는 각각 부모 테이블을 참조하는 FK
   - 문제 상황 : 비즈니스 규칙을 다시 확인 - '하나의 주문에는 동일한 상품이 중복으로 들어가면 안 된다'
      + 예를 들어, '주문번호 100번'에 '청바지' 상품이 두 번 기록되면 안 됨 : 데이터가 중복으로 쌓이면 나중에 통계를 낼 때나 재고를 계산할 때 심각한 오류를 유발할 수 있음
      + 하지만 우리가 방금 설계한 order_item 테이블은 order_item_id라는 대리 키를 PK로 사용하기 때문에, order_id와 product_id의 조합이 중복되어도 데이터베이스는 오류를 발생시키지 않고 그대로 INSERT를 허용
      + 이것은 잠재적인 데이터 부정합 문제로 이어질 수 있음

   - 중복 데이터 입력 예시
      + order_item 테이블의 현재 구조에서는 order_item_id만 유일성이 보장
      + 따라서 order_id와 product_id가 완전히 동일한 데이터가 여러 번 삽입될 수 있음
      + 100번 주문에 1번 상품(청바지)을 두 번 추가
```sql
-- 상품 데이터 삽입
INSERT INTO product_non_identifying (product_id, name, price)
VALUES(1, '청바지', 50000);
INSERT INTO product_non_identifying (product_id, name, price)
VALUES(2, '티셔츠', 25000);

-- 주문 100 생성
INSERT INTO orders_non_identifying (order_id, order_date)
VALUES(100, '2025-09-01');

-- 100번 주문에 1번 상품(청바지) 추가
INSERT INTO order_item_non_identifying(order_id, product_id, order_price, order_quantity)
VALUES (100, 1, 30000, 1);

-- 실수로 동일한 상품을 한 번 더 추가
INSERT INTO order_item_non_identifying (order_id, product_id, order_price, order_quantity)
VALUES (100, 1, 30000, 1);
```

   - 이제 테이블을 조회해서 결과를 확인
```sql
SELECT *
FROM order_item_non_identifying
WHERE order_id = 100;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/4120ab7c-141a-4fa0-8abe-4a7e24f1251f">
</div>

  - 결과를 보면 order_item_id는 1과 2로 서로 다르기 때문에 데이터베이스는 아무런 문제 없이 데이터를 추가
  - 하지만 order_id와 product_id 는 (100, 1)로 완전히 중복
    + 이 테이블의 데이터만 보면 100번 고객은 청바지를 두 번 주문한 셈이 되어버림
    + 이것이 바로 우리가 해결해야 할 데이터 정합성 문제
  - 해결 방안 : UNIQUE 제약조건 추가
    + 바로 PK와는 별개로, 비즈니스적으로 중복되면 안 되는 컬럼 조합에 UNIQUE 제약조건을 추가
테이블 설계
```sql
DROP TABLE IF EXISTS order_item_non_identifying;

CREATE TABLE order_item_non_identifying (
     order_item_id BIGINT NOT NULL AUTO_INCREMENT, -- 독립적인 PK
     order_id BIGINT NOT NULL, -- 일반 컬럼 + FK
     product_id BIGINT NOT NULL, -- 일반 컬럼 + FK
     order_price INT NOT NULL,
     order_quantity INT NOT NULL,

     PRIMARY KEY (order_item_id),

     UNIQUE KEY uq_order_item (order_id, product_id), -- 이 조합은 유일해야 함

     CONSTRAINT fk_oi_non_orders FOREIGN KEY (order_id)
     REFERENCES orders_non_identifying(order_id),
     CONSTRAINT fk_oi_non_product FOREIGN KEY (product_id)
     REFERENCES product_non_identifying(product_id)
);
```
   - order_item 테이블은 비즈니스 의미가 없는 대리 키 order_item_id를 PK로 사용
     + order_id 와 product_id 는 각각 부모 테이블을 참조하는 FK
   - UNIQUE KEY uq_order_item (order_id, product_id) : 이 부분이 핵심
     + PRIMARY KEY와는 별도로 order_id와 product_id의 조합이 테이블 내에서 항상 유일한 값을 갖도록 데이터베이스 레벨에서 강제
     + 만약 중복된 조합의 데이터 삽입을 시도하면, 데이터베이스는 즉시 오류를 발생시켜 데이터의 정합성을 지켜줌
   - 장점
     + 유연성과 데이터 정합성을 모두 확보 : 비식별 관계의 장점인 단순하고 유연한 구조는 그대로 가져가면서, 식별 관계의 장점인 데이터 정합성 보장(중복 방지)까지 UNIQUE 제약조건을 통해 얻을 수 있음
       + 실무에서 가장 권장되는 방식
     + ORM 친화적 : ORM에서 OrderItem 이라는 엔티티를 다른 엔티티들과 동일한 방식으로 단순하게 다룰 수 있음

6. UNIQUE 제약조건 동작 확인
   - UNIQUE 제약조건이 추가된 order_item 테이블에 이전과 동일하게 중복 데이터를 삽입
   - 정상 데이터 추가
```sql
-- 100번 주문에 1번 상품(청바지) 추가
INSERT INTO order_item_non_identifying (order_id, product_id, order_price, order_quantity)
VALUES (100, 1, 30000, 1);
```
   - 실행 결과 : 성공적으로 1개의 행이 추가
   - 중복 데이터 추가 시도
```sql
-- 100번 주문에 1번 상품(청바지)을 중복 추가 시도
INSERT INTO order_item_non_identifying(order_id, product_id, order_price, order_quantity)
VALUES (100, 1, 30000, 1);
```
   - 실행 결과 : 데이터베이스는 이 요청을 거부하고 UNIQUE 제약조건 위반 오류를 발생
```
-- MySQL 오류 메시지 예시
Error Code: 1062. Duplicate entry '100-1' for key
'order_item_non_identifying.uq_order_item'
```
   - 오류 메시지를 보면 PK(기본 키)가 아닌 uq_order_item라는 이름의 UNIQUE KEY 제약조건 때문에 중복이 발생했음
   - 이처럼 대리 키를 PK로 사용하는 비식별 관계에서도 UNIQUE 제약조건을 활용하면, 비즈니스 규칙에 따른 데이터의 정합성을 완벽하게 보장할 수 있음
   - 이것이 바로 M:N 관계를 모델링할 때 가장 권장되는 현대적인 설계 방식 
