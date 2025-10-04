-----
### 조회 실습 데이터 준비
-----
1. 모든 데이터 분석과 조회의 첫걸음은 조회할 데이터가 '존재'해야 한다는 것
2. 데이터베이스 변경
```sql
USE my_shop;
```

3. 샘플 데이터 입력
  - 예제를 깔끔하게 실행하기 위해서 먼저 앞서 생성한 테이블을 초기화
```sql
SET FOREIGN_KEY_CHECKS = 0; -- 비활성화

truncate table products;
truncate table customers;
truncate table orders;

SET FOREIGN_KEY_CHECKS = 1; -- 활성화
```

  - 다음 SQL 쿼리는 테이블에 실제 데이터를 삽입하는 INSERT 문
  - 고객 데이터 삽입 (customers)
```sql
INSERT INTO customers (name, email, password, address, join_date) VALUES
('이순신', 'yisunsin@example.com', 'password123', '서울특별시 중구 세종대로', '2023-05-01'),
('세종대왕', 'sejong@example.com', 'password456', '서울특별시 종로구 사직로', '2024-05-01'),
('장영실', 'youngsil@example.com', 'password789', '부산광역시 동래구 복천동', '2025-05-01');
```

  - 상품 데이터 삽입 (products)
```sql
INSERT INTO products (name, description, price, stock_quantity) VALUES
('갤럭시', '최신 AI 기능이 탑재된 고성능 스마트폰', 10000, 55),
('LG 그램', '초경량 디자인과 강력한 성능을 자랑하는 노트북', 20000, 35),
('아이폰', '직관적인 사용자 경험을 제공하는 스마트폰', 5000, 55),
('에어팟', '편리한 사용성의 무선 이어폰', 3000, 110),
('보급형 스마트폰', NULL, 5000, 100);
```

  - 주문 데이터 삽입 (orders)
```sql
INSERT INTO orders (customer_id, product_id, quantity) VALUES
(1, 1, 1), -- 이순신 고객이 갤럭시 1개 주문
(2, 2, 1), -- 세종대왕 고객이 LG 그램 1개 주문
(3, 3, 1), -- 장영실 고객이 아이폰 1개 주문
(1, 4, 2), -- 이순신 고객이 에어팟 2개 추가 주문
(2, 2, 1); -- 세종대왕 고객이 LG 그램 1개 주문 (추가 주문)
```

4. 샘플 데이터 확인 : 다음 SQL을 실행해서 데이터가 잘 입력되었는지 확인
```sql
SELECT * FROM customers;

SELECT * FROM products;

SELECT * FROM orders;
```

  - SELECT * FROM customers; 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/e374a0c2-1d6b-492c-9fc6-1054cdf5fc74">
</div>

   - customer_id는 자동으로 1부터 증가

  - SELECT * FROM products; 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/6a188993-553f-4755-aef8-d386f78b9450">
</div>

   - product_id는 자동으로 1부터 증가
     
  - SELECT * FROM orders; 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/83150d6d-d674-40b1-8258-12d6c38e2777">
</div>

  - order_id는 자동으로 1부터 증가
  - order_date는 DEFAULT 설정에 따라 현재 시각이 기록되며, status는 DEFAULT 설정에 따라 '주문접수'로 표시
