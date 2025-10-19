-----
### 제2 정규형
-----
1. orders_1nf를 보니 상품의 가격(product_pric )은 있는데, 주문 시점의 가격 스냅샷인 주문 가격 order_price)이 없음 : 주문 가격 필드도 추가
<div align="center">
<img src="https://github.com/user-attachments/assets/7e28c000-d9b7-49a5-9ee4-b54651c72733">
</div>

2. 제 2 정규형이 필요한 이유
   - orders_1nf 테이블 : 이 테이블의 기본 키(PK)는 (order_id, product_id) 복합키
     + 즉, 주문 ID와 상품 ID가 합쳐져야 하나의 행을 고유하게 식별할 수 있음

   - 여기서 함수 종속 관계를 분석 
     + 기본 키 모두에 종속되는 컬럼들
       * (order_id, product_id) → order_price
       * (order_id, product_id) → order_quantity
       * order_price와 order_quantity는 기본 키 전체에 종속 (올바른 관계)
     + 기본 키의 일부에만 종속되는 컬럼들
       * order_id → member_id
       * order_id → member_name
       * order_id → ordered_at
         * member_id, member_name, ordered_at는 기본 키의 일부인 order_id에만 종속 (부분 함수 종속)
         * product_id와는 아무런 관련이 없음
       * product_id → product_name, product_price
         * product_name, product_price는 기본 키의 일부인 product_id에만 종속 (부분 함수 종속)

   - 💡 이처럼, 기본 키의 일부에만 종속되는 컬럼이 존재하는 것을 부분 함수 종속(Partial Functional Dependency)
   - 부분 함수 종속은 다음과 같은 데이터 중복 및 이상 현상을 일으킴
       + 데이터 중복 : 1001번 주문에 상품이 10개 포함된다면, member_name('션')과 ordered_at는 10번 반복 저장되어야 하며, 이는 심각한 공간 낭비
       + 갱신 이상 : '션' 회원이 개명이라도 하면, 관련된 모든 주문-상품 데이터를 찾아 이름을 수정해야 함
       + 삽입/삭제 이상 : 제1 정규형에서 보았던 이상 현상이 여전히 일부 남아있음

3. 제2 정규형의 정의와 적용
   - 💡 제2 정규형(2NF)
      + 제1 정규형을 만족해야 함
      + 테이블의 모든 컬럼이 기본 키에 대해 완전 함수 종속(Fully Functional Dependent)이어야 함
        * 즉, 부분 함수 종속이 없어야 함

   - 부분 함수 종속을 제거하는 방법 : 종속 관계에 맞게 테이블을 분리
      + order_id 에만 종속되는 컬럼들 (member_id, member_name, ordered_at)을 모아 orders 테이블을 생성
      + product_id 에만 종속되는 컬럼들 (product_name, product_price)을 모아 product 테이블을 생성다      
      + (order_id, product_id) 기본 키 전체에 종속되는 컬럼들 (order_quantity , order_price)을 모아 연결 테이블인 order_item 테이블 생성

4. 단계적 설명 및 예제
   - orders_1nf 테이블을 3개의 테이블로 분리
```sql
DROP TABLE IF EXISTS order_item_2nf;
DROP TABLE IF EXISTS orders_2nf;
DROP TABLE IF EXISTS product_2nf;

-- 1. orders 테이블 생성 (주문 정보)
CREATE TABLE orders_2nf (
   order_id INT NOT NULL,
   member_id INT NOT NULL,
   member_name VARCHAR(50) NOT NULL, -- 아직 3NF 위반 요소를 남겨둠
   ordered_at DATETIME NOT NULL,

   PRIMARY KEY (order_id)
);

-- 2. product 테이블 생성 (상품 정보)
CREATE TABLE product_2nf (
     product_id INT NOT NULL,
     product_name VARCHAR(100) NOT NULL,
     product_price INT NOT NULL,

     PRIMARY KEY (product_id)
);

-- 3. order_item 테이블 생성 (주문-상품 연결 정보)
CREATE TABLE order_item_2nf (
     order_id INT NOT NULL,
     product_id INT NOT NULL,
     order_price INT NOT NULL,
     order_quantity INT NOT NULL,

     PRIMARY KEY (order_id, product_id),

     FOREIGN KEY (order_id) REFERENCES orders_2nf (order_id),
     FOREIGN KEY (product_id) REFERENCES product_2nf (product_id)
);
```

   - 분리된 테이블에 맞추어 데이터를 삽입
```sql
-- product_2nf 데이터 삽입
INSERT INTO product_2nf (product_id, product_name, product_price) VALUES
(10, '노트북', 1500000),
(15, '키보드', 50000),
(20, '마우스', 30000);

-- orders_2nf 데이터 삽입
INSERT INTO orders_2nf (order_id, member_id, member_name, ordered_at) VALUES
(1001, 1, '션', '2025-08-20 10:00:00'),
(1002, 2, '네이트', '2025-08-21 11:00:00'),
(1003, 1, '션', '2025-08-21 12:00:00');

-- order_item_2nf 데이터 삽입
INSERT INTO order_item_2nf (order_id, product_id, order_price, order_quantity)
VALUES
(1001, 10, 1500000, 2),
(1001, 15, 50000, 1),
(1002, 10, 1500000, 1),
(1003, 20, 30000, 1);
```

   - 실행 결과 확인
```sql
SELECT *
FROM orders_2nf;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e5b15afd-e0cf-4a06-81e0-d6cdf5274ac4">
</div>

<img width="610" height="218" alt="image" src="" />


   - 실행 결과 확인
```sql
SELECT *
FROM product_2nf;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/35445460-4302-49ec-af20-0216e4ea7280">
</div>

   - 실행 결과 확인
```sql
SELECT *
FROM order_item_2nf;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/be377321-0161-487e-b33c-265e1ea0eb35">
</div>

   - 이제 부분 함수 종속이 사라졌음
   - orders_2nf 테이블의 데이터 중복(주문 날짜, 회원 이름)도 orders_1nf 테이블에 비해 훨씬 줄어들었음
   - 하지만 orders_2nf 테이블을 자세히 보면, 여전히 갱신 이상의 문제가 남아있음 : '션' 회원의 이름이 두 번이나 중복 저장되어 있음
   - 이 문제를 해결하는 것이 제 3 정규화
