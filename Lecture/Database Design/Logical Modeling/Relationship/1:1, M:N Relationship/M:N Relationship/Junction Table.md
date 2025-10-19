-----
### 다대다(M:N) 관계 - 연결 테이블
-----
1. 해결책: 연결 테이블(Junction Table)
   - 다대다 관계를 푸는 표준적인 해법은 중간에 새로운 테이블을 하나 더 만들어서, 기존의 다대다 관계를 두 개의 일대다 관계로 풀어내는 것
   - 이 중간 테이블을 연결 테이블(Junction Table) 또는 개념적 모델링에서는 연관 엔티티(Associative Entity)라고 부름
   - 다대다 관계 해법 : 다대다는 중간 테이블을 두어 1:N , N:1 관계로 풀어냄

2. 쇼핑몰의 orders와 product 관계를 예로 들면, order_product라는 연결 테이블을 만드는 것
   - 참고로 여기서는 orders와 product를 연결하는 의미로 order_product 라는 이름을 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/be4717a4-2083-43be-963b-a07b32d13f03">
</div>

   - orders와 product의 관계 (M:N)
     + 기존 M:N 관계 : orders (M) <---> (N) product
     + 연결 테이블로 분해 : orders (1) <---> (N) order_product (N) <---> (1) product
   - 다대다 관계가 두 개의 일대다 관계로 나뉨
     + 하나의 주문(orders)은 여러 주문 상품(order_product)을 가질 수 있음 (1:N)
     + 하나의 상품(product)은 여러 주문 상품(order_product)에 포함될 수 있음 (1:N)
   - 이렇게 하면 관계형 데이터베이스가 완벽하게 다대다 관계를 이해하고 처리할 수 있는 구조가 됨
   - 다대다 관계
<div align="center">
<img src="https://github.com/user-attachments/assets/639d465d-5bf6-44fd-88e8-df742f73ae8a">
</div>

   - 연결 테이블 - 일대다, 다대일 관계
<div align="center">
<img src="https://github.com/user-attachments/assets/ffbdfe13-4f7d-4538-a330-67b014e9dcb6">
</div>

3. 연결 테이블 예시
   - orders, product, 그리고 이 둘을 잇는 연결 테이블 order_product를 직접 생성 : 이를 통해 어떻게 다대다 관계가 두 개의 일대다 관계로 해소되는지 눈으로 직접 확인
   - 테이블 생성
      + 먼저 예제에 필요한 product, orders, order_product 세 개의 테이블을 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/d90a80ca-76a0-4235-ac60-3daa9b384f02">
</div>

```sql
DROP TABLE IF EXISTS order_item; -- 다른 예제 충돌 예방
DROP TABLE IF EXISTS order_product;
DROP TABLE IF EXISTS product;
DROP TABLE IF EXISTS orders;

-- 상품 테이블 생성
CREATE TABLE product (
   product_id BIGINT NOT NULL, -- 상품id 직접 입력
   name VARCHAR(100) NOT NULL,
   price INT NOT NULL,
   PRIMARY KEY (product_id)
);

-- 주문 테이블 생성
CREATE TABLE orders (
   order_id BIGINT NOT NULL, -- 주문id 직접 입력
   order_date DATE,
   PRIMARY KEY (order_id)
);

-- 연결 테이블(주문-상품) 생성
CREATE TABLE order_product (
   order_product_id BIGINT NOT NULL AUTO_INCREMENT,
   order_id BIGINT NOT NULL, -- orders 테이블의 FK
   product_id BIGINT NOT NULL, -- product 테이블의 FK

   PRIMARY KEY (order_product_id),
   -- 한 주문에 동일한 상품이 중복으로 들어가는 것을 방지
   CONSTRAINT uq_order_product UNIQUE (order_id, product_id),
   CONSTRAINT fk_order_product_orders FOREIGN KEY (order_id)
   REFERENCES orders (order_id),
   CONSTRAINT fk_order_product_product FOREIGN KEY (product_id)
   REFERENCES product (product_id)
);
```
   - order_product 테이블이 핵심 : 이 테이블은 orders의 기본 키인 order_id와 product의 기본 키인 product_id 를 각각 외래 키(FK)로 가짐
     + 이 두 외래 키가 orders 와 product를 연결하는 다리 역할
   - CONSTRAINT uq_order_product UNIQUE (order_id, product_id) : 유니크 제약조건을 통해 같은 주문에 같은 상품이 중복 선택되지 않도록 막음
  
4. 데이터 삽입
   - 다음과 같은 두 가지 주문 시나리오를 가정
      + 주문 100 (order_id: 100) : '청바지'와 '티셔츠'를 함께 주문
      + 주문 101 (order_id: 101) : '청바지'를 주문
      + 이 시나리오를 보면 '청바지'는 여러 주문(주문 100, 주문 101)에 포함되고, '주문 100'은 여러 상품('청바지', '티셔츠')을 포함하는 다대다 관계임을 명확히 알 수 있음
```sql
-- 상품 데이터 삽입
INSERT INTO product(product_id, name, price) VALUES(1, '청바지', 50000);
INSERT INTO product(product_id, name, price) VALUES(2, '티셔츠', 25000);

-- 주문 100 생성
INSERT INTO orders (order_id, order_date)
VALUES(100, '2025-09-01');

-- 연결 테이블 데이터 삽입
-- 주문 100에 대한 상품 정보
INSERT INTO order_product (order_id, product_id)
VALUES(100, 1); -- 주문 100번, 청바지
INSERT INTO order_product (order_id, product_id)
VALUES(100, 2); -- 주문 100번, 티셔츠

-- 주문 101 생성
INSERT INTO orders (order_id, order_date)
VALUES(101, '2025-09-02');

-- 주문 101에 대한 상품 정보
INSERT INTO order_product (order_id, product_id)
VALUES(101, 1); -- 주문 101번, 청바지 1개
```
   - 결과 확인 : 데이터 삽입 후 각 테이블의 내용을 조회
   - product 테이블
```sql
SELECT *
FROM product;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a8ae9b3f-e89f-4e67-aa17-9dcb62516a3a">
</div>

   - orders 테이블
```sql
SELECT *
FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/bcc2363b-c29c-44c6-aa32-a4c1d5923019">
</div>

   - order_product 테이블
```sql
SELECT *
FROM order_product;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/6a623a62-ab31-40e4-8fff-0cf5f45d2b81">
</div>

   - order_product 테이블 : 이 테이블이 다대다 관계의 모든 정보를 담고 있음
   - order_id가 100인 주문은 product_id 1번('청바지')과 2번('티셔츠')을 포함 (orders : order_product = 1 : N)
<div align="center">
<img src="https://github.com/user-attachments/assets/8369fe07-7803-40bd-bb87-69cd516b1a52">
</div>

   - product_id가 1인 상품('청바지')은 order_id 100번과 101번 주문에 모두 포함 (product : order_product = 1 : N)
<div align="center">
<img src="https://github.com/user-attachments/assets/19f70c20-360c-4f29-b845-47fbec5fe7a5">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/baf9d33d-8c20-48d9-9480-6f6c5ae8a348">
</div>

5. 연결 테이블과 조인
   - 주문 100번에 대한 상품 목록 조회 : orders, order_product, product 세 테이블을 조인해서 100번 주문에 포함된 상품들의 이름과 가격을 조회
```sql
SELECT
   o.order_id,
   p.name,
   p.price
FROM orders o
JOIN order_product op ON o.order_id = op.order_id
JOIN product p ON op.product_id = p.product_id
WHERE o.order_id = 100;
```
   - FROM orders o : 조인의 시작점을 orders 테이블로 잡고, o 라는 별칭(alias)을 붙임
   - JOIN order_product op ON o.order_id = op.order_id : orders 테이블과 order_product 테이블을 order_id를 기준으로 연결하여, 이때 일대다 관계의 조인이 발생 (조인 뻥튀기 발생)
   - JOIN product p ON op.product_id = p.product_id : 위에서 연결된 결과에 다시 product 테이블을 product_id를 기준으로 연결하여, 이때 다대일 관계의 조인이 발생
   - WHERE o.order_id = 100 : 이렇게 3개의 테이블이 모두 연결된 거대한 가상의 테이블에서 order_id가 100인 데이터만 필터링

<div align="center">
<img src="https://github.com/user-attachments/assets/f04b17cb-2e81-4779-9699-986603b6b72e">
</div>

   - 결과를 보면, 연결 테이블을 통해 100번 주문이 '청바지'와 '티셔츠' 두 상품과 관계를 맺고 있다는 사실을 명확하게 확인할 수 있음
   - '청바지'가 포함된 주문 목록 조회 : 이번에는 반대로 '청바지'(product_id = 1)가 포함된 모든 주문을 조회
```sql
SELECT
     p.name,
     o.order_id,
     o.order_date
FROM product p
JOIN order_product op ON p.product_id = op.product_id
JOIN orders o ON op.order_id = o.order_id
WHERE p.product_id = 1;
```
   - 이번에는 product 테이블에서 조회를 시작
   - JOIN order_product op ON p.product_id = op.product_id : 일대다 조인 (조인 뻥튀기 발생)
   - JOIN orders o ON op.order_id = o.order_id : 다대일 조인
<div align="center">
<img src="https://github.com/user-attachments/assets/368b1313-92f9-4043-a222-decfaf4426bf">
</div>

   - 이처럼 연결 테이블(order_product)이 있었기 때문에, 특정 상품이 어떤 주문들에 포함되어 있는지도 쉽게 조회 가능
   - 이것이 바로 관계형 데이터베이스가 관계를 다루는 방식 : 다대다 관계를 직접 표현할 수 없으므로, 중간에 연결 테이블을 두어 두 개의 일대다 관계로 풀어내는 것
   - 이처럼 연결 테이블 order_product를 통해 기존의 복잡했던 다대다 관계가 두 개의 명확한 일대다 관계로 완벽하게 해결    
   - 이렇게 중간에 연결 테이블을 두는 방식이 관계형 데이터베이스에서 다대다 관계를 다루는 표준적인 방법

6. 연결 테이블의 본질 : 관계를 모델링
   - 그렇다면 연결 테이블은 근본적으로 어떻게 다대다 관계의 문제를 해결하는 것일까? 단순히 테이블을 하나 추가했기 때문일까?
     + 핵심은 관계 자체를 하나의 독립된 데이터로 보고, 그것을 테이블로 모델링했다는 데 있음

   - 다대다 관계
<div align="center">
<img src="https://github.com/user-attachments/assets/1da99457-2332-4bef-9e46-8752d319fd20">
</div>

   - 연결 테이블 : 일대다, 다대일 관계
<div align="center">
<img src="https://github.com/user-attachments/assets/b4cdd636-03f0-40ac-b9eb-1f9479a64cd7">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/34f7e1ee-ae65-452c-b0bf-3dba545cbb86">
</div>

   - order_product 테이블의 한 행(row)이 무엇을 의미하는지 생각
<div align="center">
<img src="https://github.com/user-attachments/assets/ed5da9d1-ab9d-4ac1-85ca-5433629be79a">
</div>

   - order_product 테이블의 첫 번째 행 (order_id:100, product_id:1) : '100번 주문은 1번 상품(청바지)을 포함한다'라는 하나의 사실 또는 사건을 의미
   - 두 번째 행 (order_id:100, product_id:2) : '100번 주문은 2번 상품(티셔츠)을 포함한다'라는 또 다른 하나의 사실을 의미
   - 즉, 연결 테이블은 orders와 product 사이에서 발생할 수 있는 수많은 관계의 경우의 수를 하나하나의 독립된 데이터로 저장하는 공간
   - 이렇게 관계를 구체적인 데이터로 만들어 테이블에 담는 순간, 기존의 복잡했던 다대다(M:N) 관계는 아주 명확한 두 개의 일대다(1:N) 관계로 자연스럽게 해소

7. orders와 order_product의 관계 (1:N)
   - 하나의 주문(orders)은 여러 개의 주문-상품 내역(order_product)을 가질 수 있음
   - order_id 100번 주문은 order_product 테이블에서 2개의 행(청바지, 티셔츠)을 가짐
   - 이는 완벽한 일대다(1:N) 관계이며, '다(N)' 쪽에 외래 키(order_id)가 있으므로 아무런 문제가 없음

8. product 와 order_product 의 관계 (1:N)
   - 하나의 상품(product)은 여러 개의 주문-상품 내역(order_product)에 포함될 수 있음
   - product_id 1번 상품(청바지)은 order_product 테이블에서 2개의 행(100번 주문, 101번 주문)을 가짐
   - 이 또한 완벽한 일대다(1:N) 관계이며, '다(N)' 쪽에 외래 키( product_id )가 있으므로 아무런 문제가 없음

9. 결론적으로, 연결 테이블은 두 엔티티 사이의 추상적인 '관계'를 '구체적인 데이터'로 변환하는 것
    - 이 변환을 통해 관계형 데이터베이스가 가장 잘 처리할 수 있는 단순한 일대다 관계 두 개로 문제를 바꾸어 해결하는 것
    - 이것이 바로 관계형 데이터베이스에서 다대다 관계를 해결하는 방식
