-----
### 문제와 풀이
-----
1. 문제 1 : 고객의 기본 정보만 보여주는 뷰 생성하기
   - users 테이블에는 고객의 주소, 생년월일과 같은 민감한 정보가 포함되어 있음
   - 마케팅팀에서는 고객에게 이메일을 보내기 위해 이름과 이메일 주소만 필요
   - 보안을 위해 고객의 user_id, name, email 컬럼만을 포함하는 v_customer_email_list 라는 이름의 뷰(View)를 생성

<div align="center">
<img src="https://github.com/user-attachments/assets/0f4f0a40-582e-4d7c-8f0f-be7759443636">
</div>

```sql
-- 기존에 뷰가 있다면 삭제
DROP VIEW IF EXISTS v_customer_email_list;

-- 뷰 생성
CREATE VIEW v_customer_email_list
AS
SELECT
   user_id,
   name AS 고객명,
   email AS 이메일
FROM
   users;

-- 생성된 뷰 조회
SELECT *
FROM v_customer_email_list;
```

2. 문제 2 : 주문 상세 정보를 통합한 뷰 생성하기
   - 운영팀에서는 현재 주문 내역을 확인할 때마다 orders, users, products 테이블을 매번 JOIN 해야 해서 불편함을 겪고 있음
   - 주문 ID(order_id), 고객 이름(name), 상품 이름(name), 주문 수량(quantity), 주문 상태(status)를 한 번에 볼 수 있는 v_order_summary 라는 이름의 뷰를 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/faef11c3-2796-436e-a39e-75da2fe3eed2">
</div>

```sql
-- 기존에 뷰가 있다면 삭제
DROP VIEW IF EXISTS v_order_summary;

-- 뷰 생성
CREATE VIEW v_order_summary
AS
SELECT
   o.order_id,
   u.name AS '고객명',
   p.name AS '상품명',
   o.quantity AS '주문수량',
   o.status AS '주문상태'
FROM
   orders o
JOIN
   users u ON o.user_id = u.user_id
JOIN
   products p ON o.product_id = p.product_id;

-- 생성된 뷰 조회
SELECT *
FROM v_order_summary;
```

3. 문제 3 : '전자기기' 카테고리 매출 분석 뷰 생성하기
   - 전략기획팀은 '전자기기' 카테고리의 상품들에 대한 주문 건수와 총매출액을 집중적으로 분석하고자 함
   - products 테이블과 orders 테이블을 JOIN 하여, '전자기기' 카테고리에 속한 상품들의 총 주문 건수(total_orders)와 총 매출액(total_sales)을 계산하는 v_electronics_sales_status 뷰를 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/f791ca77-cb37-4121-a9fc-57f0c1e9737f">
</div>

```sql
-- 기존에 뷰가 있다면 삭제
DROP VIEW IF EXISTS v_electronics_sales_status;

-- 뷰 생성
CREATE VIEW v_electronics_sales_status
AS
SELECT
   p.category,
   COUNT(o.order_id) AS total_orders,
   SUM(p.price * o.quantity) AS total_sales
FROM
   orders o
JOIN
   products p ON o.product_id = p.product_id
WHERE
   p.category = '전자기기'
GROUP BY
   p.category;

-- 생성된 뷰 조회
SELECT *
FROM v_electronics_sales_status;
```

4. 문제 4: 기존 뷰에 정보 추가하여 수정하기
   - 문제 3에서 만들었던 v_electronics_sales_status 뷰가 매우 유용하다는 피드백을 받음
   - 전략기획팀에서 총 매출액뿐만 아니라, 평균 주문액(average_order_value)도 함께 보고 싶다는 추가 요청이 발생
   - 기존 뷰를 삭제하지 말고, ALTER VIEW를 사용하여 v_electronics_sales_status 뷰의 정의를 수정
   - 총 주문 건수, 총매출액, 그리고 평균 주문액을 보여주도록 변경해야 함

<div align="center">
<img src="https://github.com/user-attachments/assets/b37cf7c2-d1f3-4f41-9082-d48e02b77c22">
</div>

  ```sql
-- 뷰 수정
ALTER VIEW v_electronics_sales_status
AS
SELECT
   p.category,
   COUNT(o.order_id) AS total_orders,
   SUM(p.price * o.quantity) AS total_sales,
   AVG(p.price * o.quantity) AS average_order_value
FROM
   orders o
JOIN
   products p ON o.product_id = p.product_id
WHERE
   p.category = '전자기기'
GROUP BY
   p.category;

-- 수정된 뷰 조회
SELECT *
FROM v_electronics_sales_status;
```
