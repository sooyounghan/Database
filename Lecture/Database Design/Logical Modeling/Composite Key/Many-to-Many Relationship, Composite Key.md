-----
### 다대다 관계와 복합키
-----
1. 자연 키가 아닌, 다른 테이블의 대리 키(외래 키)들을 조합해서 복합키를 만드는 경우는 어떨까?
   - 이는 실무에서 아주 흔하게 마주치는 상황이며, 특히 다대다(N:M) 관계를 해결할 때 나타남

2. 문제 상황 : 다대다 관계 모델링
   - 우리의 쇼핑몰에서 '주문'과 '상품'의 관계를 다시 생각 
      + 하나의 주문에는 여러 개의 상품이 포함될 수 있음 (1:N)
      + 하나의 상품은 여러 주문에 포함될 수 있음 (1:N)
   - 이처럼 관계의 양쪽 모두가 다(N)의 관계를 가질 때, 이를 다대다(N:M) 관계라고 부름

3. 관계형 데이터베이스에서는 테이블 간의 관계를 외래 키로 표현하는데, 다대다 관계는 두 테이블만으로는 직접 표현할 수 없음
   - orders 테이블에 product_id 컬럼을 추가하면 한 주문에는 상품 하나만 담을 수 있게 되고, 반대로 product 테이블에 order_id 컬럼을 넣으면 하나의 상품은 단 하나의 주문에만 속하게 되는 문제가 발생함
   - 다대다 관계는 개념적 모델링 단계에서 연관 엔티티를 통해 해결할 수 있음

4. 다대다 관계 해소를 위한 연결 테이블
   - 이 문제를 해결하기 위해, 두 테이블의 중간에서 다리 역할을 하는 연결 테이블(연관 엔티티)을 만듬
   - 쇼핑몰 모델에서는 주문 항목 (order_item) 테이블이 바로 이 역할을 수행
     + 이 연결 테이블을 통해 주문과 상품의 다대다 관계를 두 개의 일대다(1:N) 관계로 풀어낼 수 있음
     + orders (1) - (N) order_item (N) - (1) product
       * orders (1) : order_item (N)
       * product (1) : order_item (N)
   - order_item 테이블은 orders 테이블의 기본 키인 order_id와 product 테이블의 기본 키인 product_id를 외래 키(FK)로 받아서 두 테이블의 관계를 맺어줌
   - order_item 테이블의 기본 키(PK)는 무엇으로 해야하는가?
     + 방법 1 : {order_id, product_id} 를 복합 기본 키로 사용
     + 방법 2 : order_item_id 라는 별도의 대리 키를 기본 키로 사용하고, {order_id, product_id}에는 UNIQUE 제약조건을 설정

5. 방법 1 : {order_id, product_id} 를 복합 기본 키로 사용하기
   - 이 방법은 가장 직관적이고 또 전통적인 다대다 관계 모델링에서 선택하는 방식
   - order_item의 한 행은 "어떤 주문에 어떤 상품이 포함되었는가"를 나타냄 : 따라서 {주문 ID, 상품 ID} 의 조합 자체가 이 행을 식별하는 가장 자연스러운 식별자가 됨
   - 테이블 구조 및 데이터 예시 (SQL)
      + 전체 예제를 실행할 수 있도록 product_c, orders_c, order_item_c 테이블을 순서대로 생성하고 데이터를 삽입
```sql
-- 테이블 초기화
DROP TABLE IF EXISTS order_item_c;
DROP TABLE IF EXISTS orders_c;
DROP TABLE IF EXISTS product_c;

-- 상품 테이블 생성
CREATE TABLE product_c (
   product_id BIGINT NOT NULL AUTO_INCREMENT,
   name VARCHAR(100) NOT NULL,
   price INT NOT NULL,
   PRIMARY KEY (product_id)
);

-- 주문 테이블 생성
CREATE TABLE orders_c (
   order_id BIGINT NOT NULL AUTO_INCREMENT,
   ordered_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
   PRIMARY KEY (order_id)
);

-- 주문 항목 테이블 생성 (복합키 사용)
CREATE TABLE order_item_c (
   order_id BIGINT NOT NULL, -- 주문 ID (PK, FK)
   product_id BIGINT NOT NULL, -- 상품 ID (PK, FK)
   order_price INT NOT NULL, -- 주문 당시 가격
   count INT NOT NULL, -- 주문 수량

   PRIMARY KEY (order_id, product_id), -- 복합 기본 키

   CONSTRAINT fk_order_item_c_orders FOREIGN KEY (order_id)
   REFERENCES orders_c (order_id),

   CONSTRAINT fk_order_item_c_product FOREIGN KEY (product_id)
   REFERENCES product_c (product_id)
);
```
```sql
-- 샘플 데이터 삽입
INSERT INTO product_c (product_id, name, price) VALUES (101, '노트북', 1500000);
INSERT INTO product_c (product_id, name, price) VALUES (102, '마우스', 20000);
INSERT INTO orders_c (order_id) VALUES (1);

-- 1번 주문에 101번 상품 2개 추가
INSERT INTO order_item_c (order_id, product_id, order_price, count)
VALUES (1, 101, 1500000, 2);

-- 1번 주문에 102번 상품 1개 추가
INSERT INTO order_item_c (order_id, product_id, order_price, count)
VALUES (1, 102, 20000, 1);

-- (실패) 1번 주문에 101번 상품을 또 추가하려고 시도
-- INSERT INTO order_item_c (order_id, product_id, order_price, count)
-- VALUES (1, 101, 1500000, 3);
```
   - 복합키(Composite Key)를 사용하는 예시이므로 테이블을 쉽게 구분하기 위해 뒤에 _c 를 붙임
<div align="center">
<img src="https://github.com/user-attachments/assets/113f0f74-07be-42cb-be59-161dd2014a48">
</div>

   - 부모의 PK를 자신의 PK + FK로 사용하는 관계를 식별 관계라고 하고 실선을 사용
   - 실패하는 INSERT 문 실행 결과 : 주석 처리된 마지막 INSERT 문의 주석을 풀고 실행하면, 기본 키 중복 오류가 발생
```
Error Code: 1062. Duplicate entry '1-101' for key 'order_item_c.PRIMARY'
```
   - 데이터 확인 : 성공적으로 입력된 데이터를 확인
```sql
SELECT *
FROM order_item_c;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8f58ab62-ff2f-492b-8bfd-da4daf210a44">
</div>

   - 이처럼 복합키는 데이터의 정합성을 강력하게 지켜줌
   - 장점
      + 데이터 무결성 보장 : 기본 키 제약조건에 의해 {order_id, product_id} 조합의 중복이 원천적으로 차단되므로, 즉, 하나의 주문에 동일한 상품이 두 번 이상 들어가는 것을 데이터베이스 레벨에서 막아줌
      + 논리적 명확성: 테이블의 구조만 봐도 "이 테이블은 주문과 상품의 관계를 나타내는구나"라는 것을 명확히 알 수 있음
  
   - 단점
     + PK가 너무 '뚱뚱하다' (Fat Key) : 기본 키가 두 개의 BIGINT 컬럼으로 구성되어 '뚱뚱함'
       * 만약 이 order_item_c 테이블을 다른 테이블이 참조해야 하는 상황(예) 특정 주문 항목에 대한 문의사항을 저장하는 테이블)이 생긴다면, 외래 키로 이 두 개의 컬럼({order_id, product_id})을 모두 가져가야 함
       * 이는 외래 키 참조를 복잡하게 만들고 저장 공간을 낭비함

     + 확장성의 제약 : order_item_c 테이블 자체가 더 복잡한 관계의 부모 테이블이 될수록 이 '뚱뚱한' 복합키는 계속해서 하위 테이블로 전파되어 모델의 유연성을 떨어뜨림
     + ORM 사용의 불편함 : JPA와 같은 현대적인 ORM 프레임워크에서 복합키를 기본 키로 매핑하려면 별도의 식별자 클래스(@IdClass 또는 @EmbeddedId)를 만들어야 하는 등 개발의 복잡성이 증가
       * 단일 대리 키를 사용하는 것이 훨씬 편리

   - 그래서 현대적인 데이터베이스 설계에서는 다대다 관계를 위한 연결 테이블에서도 별도의 대리 키를 기본 키로 사용하는 것을 권장

6. 방법 2 : 대리 키 - PK + 복합 UNIQUE 제약조건 사용하기 (권장)
   - 이 방법은 현대적인 데이터베이스 설계에서 더 선호되는 방식
   - order_item 테이블 자체의 식별을 위한 대리 키(order_item_id)를 두고, 비즈니스적 유일성 보장은 {order_id, product_id} 에 UNIQUE 제약조건을 걸어서 해결
   - 이것이 바로 우리가 최종적으로 채택한 order_item 테이블의 구조
   - 테이블 구조 및 데이터 예시 (SQL) : 이번에는 대리 키를 사용하는 _s 테이블들로 전체 예제를 다시 구성
```sql
-- 테이블 초기화
DROP TABLE IF EXISTS order_item_s;
DROP TABLE IF EXISTS orders_s;
DROP TABLE IF EXISTS product_s;

-- 상품 테이블 생성
CREATE TABLE product_s (
   product_id BIGINT NOT NULL AUTO_INCREMENT,
   name VARCHAR(100) NOT NULL,
   price INT NOT NULL,
   PRIMARY KEY (product_id)
);

-- 주문 테이블 생성
CREATE TABLE orders_s (
   order_id BIGINT NOT NULL AUTO_INCREMENT,
   ordered_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
   PRIMARY KEY (order_id)
);

-- 주문 항목 테이블 생성 (대리 키 + UNIQUE 제약조건 사용)
CREATE TABLE order_item_s (
   order_item_id BIGINT NOT NULL AUTO_INCREMENT, -- 주문 상품 ID (PK)
   order_id BIGINT NOT NULL, -- 주문 ID (FK)
   product_id BIGINT NOT NULL, -- 상품 ID (FK)
   order_price INT NOT NULL, -- 주문 당시 가격
   count INT NOT NULL, -- 주문 수량

   PRIMARY KEY (order_item_id),
   CONSTRAINT fk_order_item_s_orders FOREIGN KEY (order_id)
   REFERENCES orders_s (order_id),
   CONSTRAINT fk_order_item_s_product FOREIGN KEY (product_id)
   REFERENCES product_s (product_id),
   UNIQUE KEY uq_order_product (order_id, product_id) -- 복합 UNIQUE 제약
);
```
```sql
-- 샘플 데이터 삽입 (예시를 위해 ID 직접 지정)
INSERT INTO product_s (product_id, name, price) VALUES (101, '노트북', 1500000);
INSERT INTO product_s (product_id, name, price) VALUES (102, '마우스', 20000);
INSERT INTO orders_s (order_id) VALUES (1);

-- 1번 주문에 101번 상품 2개 추가
INSERT INTO order_item_s (order_id, product_id, order_price, count)
VALUES (1, 101, 1500000, 2);

-- 1번 주문에 102번 상품 1개 추가
INSERT INTO order_item_s (order_id, product_id, order_price, count)
VALUES (1, 102, 20000, 1);

-- (실패) 1번 주문에 101번 상품을 중복 추가 시도
-- INSERT INTO order_item_s (order_id, product_id, order_price, count)
-- VALUES (1, 101, 1500000, 3);
```
   - 대리 키(Surrogate Key)를 사용하는 예시이므로 테이블을 쉽게 구분하기 위해 뒤에 _s를 붙임
<div align="center">
<img src="https://github.com/user-attachments/assets/75d6eb2d-aac2-4ae7-a2b4-83ab3e7f8475">
</div>

  - 실패하는 INSERT 문 실행 결과 : 마찬가지로 마지막 INSERT 문을 실행하면 UNIQUE 제약조건 위반으로 오류가 발생
```
Error Code: 1062. Duplicate entry '1-101' for key
'order_item_s.uq_order_product'
```
   - 오류 메시지에서 'PRIMARY' 키가 아닌 uq_order_product 라는 UNIQUE 키에서 중복이 발생했음을 알려줌
   - 기능적으로는 복합 기본 키와 동일한 역할을 수행하는 것을 알 수 있음
   - 데이터 확인
```sql
SELECT *
FROM order_item_s;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/41904cca-3427-43b3-b340-c5fd3c3b0cac">
</div>

   - 이 구조의 핵심
     + 기본 키는 단순하게 : 비즈니스와 무관한 order_item_id (대리 키)를 기본 키(PK)로 사용 - 이 PK는 불변하며, 작고, 다른 테이블에서 참조하기 매우 용이
     + 비즈니스 제약은 UNIQUE로 : '하나의 주문에 동일한 상품이 중복될 수 없다'는 중요한 비즈니스 규칙은 order_id와 product_id를 묶은 복합 UNIQUE 제약조건(uq_order_product)을 통해 완벽하게 보장
     
   - 장점
     + 유연하고 일관된 참조 : 다른 테이블이 order_item_s 의 특정 행을 참조할 때, order_item_id라는 단일 컬럼만 외래 키로 사용하면 되므로 구조가 단순하고 명확해짐 (예) 특정 주문 항목에 대한 문의사항을 저장하는 테이블)
     + 데이터 무결성 유지 : UNIQUE 제약조건이 복합키와 동일한 역할을 수행하여 {order_id, product_id} 조합의 중복을 막아줌
     + 개발 편의성 : order_item_id 라는 단순한 기본 키가 있으므로, JPA 같은 ORM에서 다루기가 매우 쉬우며, 복잡한 식별자 클래스 없이 일반적인 엔티티처럼 편리하게 관리할 수 있음

   - 단점
     + 약간의 오버헤드 : 기본 키 인덱스와 UNIQUE 인덱스가 별도로 생성되므로 약간의 저장 공간 및 인덱스 관리 비용이 추가될 수 있지만, 현대 하드웨어 환경에서는 거의 무시할 수 있는 수준

7. 실무 기반 데이터베이스 키 선택 전략
   - 데이터베이스 설계 시, 다대다(N:M) 관계에서도 대리 키(Surrogate Key)를 기본 키(PK)로 사용하고, 비즈니스 키 컬럼들을 복합 유니크(UNIQUE) 제약 조건으로 설정하는 방식이 가장 실용적이고 현대적인 해결책
   - 이 접근법은 복합키의 데이터 정합성 보장이라는 장점과 대리 키의 단순성, 불변성, 확장성이라는 장점을 모두 취할 수 있음
   - 애플리케이션의 비즈니스 규칙은 변할 수 있지만, 데이터베이스의 구조적 안정성은 유지되어야 함
   - 대리 키는 바로 이 안정적인 뼈대를 제공하여 다음과 같은 이점을 가져다 줌
     + 개발 복잡도 감소 : 복잡한 복합키 대신 단순한 대리 키를 사용하여 테이블 간의 관계(Join)를 단순화하고 개발 편의성을 높임
     + 유연성 및 확장성 확보 : 향후 비즈니스 로직 변경이나 컬럼 추가 시, 기본 키 자체의 변경 없이 유연하게 대응할 수 있음
     + 유지보수 용이성 : 일관되고 예측 가능한 구조는 장기적으로 시스템을 유지보수하기 쉽게 만듬

   - 이러한 이유로, (order_id, product_id) 라는 훌륭한 복합키가 존재함에도 불구하고, 현대 데이터베이스 설계에서는 order_item_id 와 같은 별도의 단일 대리 키를 기본 키로 추가하는 설계를 더 선호
   - 이것이 유지보수와 확장에 더 유리한 선택
   - 결론적으로, 데이터베이스 모델링은 애플리케이션의 전체 구조와 직결되므로, 안정성과 유연성을 고려할 때 대리 키를 기본 키로 사용하는 것이 현대 개발 환경의 표준적인 선택이라 할 수 있음
  
