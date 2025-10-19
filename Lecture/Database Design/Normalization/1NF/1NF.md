-----
### 제 1 정규형
-----
1. 제 1 정규형이 필요한 이유
   - disaster_orders 테이블의 product_info 컬럼 : '10:노트북:2:1500000,15:키보드:1:50000'와 같이 여러 상품 정보가 컬럼에 쉼표(,)로 구분되어 들어가 있음
   - 그리고 각 상품의 상세 정보 들이 : 로 구분되어 들어가 있는데, 이런 방식은 다음과 같은 심각한 문제를 야기
     + 데이터 검색의 어려움 : '키보드'가 포함된 주문을 찾으려면 모든 product_infos 문자열을 파싱해서 검색해야 함
       * 이는 매우 비효율적이며, SQL의 WHERE 절을 제대로 활용할 수 없고, 특히 인덱스를 제대로 활용할 수 없음
     + 데이터 수정의 복잡성: 1001번 주문에서 키보드 수량을 1개에서 2개로 바꾸려면, 이 복잡한 문자열을 애플리케이션에서 읽어와서 분리하고, 수량을 수정한 뒤, 다시 합쳐서 업데이트해야 함
       * 데이터베이스가 해야 할 일을 애플리케이션이 떠안게 됨
     + 데이터 타입 사용 불가 : 상품 가격, 수량 등은 숫자 타입으로 관리되어야 합계나 평균 같은 집계 함수를 쉽게 사용할 수 있음
       * 하지만 문자열 안에 섞여 있으면 불가능
   - 이런 문제를 해결하는 첫 번째 단계가 바로 제 1 정규화

2. 제 1 정규형의 정의와 적용
   - 💡 제 1 정규형(1NF) : 테이블의 모든 컬럼이 원자적(Atomic)인 값만을 가져야 함
     + '원자적'이라는 것은 더 이상 쪼갤 수 없는 값을 의미
     + product_infos 컬럼은 상품 ID, 상품명, 수량, 가격 등 여러 정보로 쪼갤 수 있으므로 원자적이지 않음
   - 제 1 정규화를 적용하기 위해, 반복되는 상품 정보를 별도의 행으로 분리해야 함
   - 단계적 설명 및 예제 : 먼저, 제1 정규화를 위반하는 테이블을 생성
```sql
DROP TABLE IF EXISTS orders_1nf_violating;

-- 제1 정규형을 위반하는 테이블 생성
CREATE TABLE orders_1nf_violating (
   order_id INT NOT NULL,
   ordered_at DATETIME NOT NULL,
   member_id INT NOT NULL,
   member_name VARCHAR(50) NOT NULL,

   product_infos VARCHAR(255) NOT NULL -- 이 컬럼이 원자성을 위반
);

-- 데이터 삽입
INSERT INTO orders_1nf_violating (order_id, ordered_at, member_id, member_name, product_infos) VALUES
(1001, '2025-08-20 10:00:00', 1, '션', '10:노트북:2:1500000,15:키보드:1:50000'),
(1002, '2025-08-21 11:00:00', 2, '네이트', '10:노트북:1:1500000'),
(1003, '2025-08-21 12:00:00', 1, '션', '20:마우스:1:30000');
```

   - 실행 결과 확인
```sql
SELECT *
FROM orders_1nf_violating;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e657fc91-7242-42ad-9c19-8a37f98d86c8">
</div>

   - 이제 이 테이블을 제1 정규형에 맞게 수정 :  product_infos 컬럼을 분리하여 각 상품이 별도의 행을 갖도록 생성
```sql
DROP TABLE IF EXISTS orders_1nf;

-- 제1 정규형을 만족하는 테이블 생성
CREATE TABLE orders_1nf (
   order_id INT NOT NULL,
   member_id INT NOT NULL,
   member_name VARCHAR(50) NOT NULL,

   product_id INT NOT NULL,
   product_name VARCHAR(100) NOT NULL,
   product_price INT NOT NULL,
   order_quantity INT NOT NULL,
   ordered_at DATETIME NOT NULL,

   PRIMARY KEY (order_id, product_id) -- 기본 키를 (order_id, product_id) 복합키로 설정
);

-- 데이터 삽입
INSERT INTO orders_1nf (order_id, member_id, member_name, product_id, product_name, product_price, order_quantity, ordered_at) VALUES
(1001, 1, '션', 10, '노트북', 1500000, 2, '2025-08-20 10:00:00'),
(1001, 1, '션', 15, '키보드', 50000, 1, '2025-08-20 10:00:00'),
(1002, 2, '네이트', 10, '노트북', 1500000, 1, '2025-08-21 11:00:00'),
(1003, 1, '션', 20, '마우스', 30000, 1, '2025-08-21 12:00:00');
```

   - 실행 결과 확인
```sql
SELECT *
FROM orders_1nf;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2aaf327a-cecf-4213-b9e9-afa3e0a54ef1">
</div>

   - 모든 컬럼이 원자적인 값을 갖게 됨
   - 하지만 여전히 데이터 중복 문제는 해결되지 않음 : 1001번 주문의 '션'회원 이름과 주문 날짜가 두 번이나 반복해서 저장되고 있음
