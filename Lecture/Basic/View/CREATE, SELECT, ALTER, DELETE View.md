-----
### 뷰 생성, 조회, 수정, 삭제
-----
1. '카테고리별 주문 현황' 쿼리를 v_category_order_status 라는 이름의 뷰로 직접 생성
2. CREATE VIEW : 뷰 생성하기
  - 뷰를 생성하는 명령어는 CREATE VIEW
```sql
CREATE VIEW 뷰이름
AS
SELECT 쿼리문;
```
   - 복잡한 쿼리를 v_category_order_status 라는 이름의 뷰로 데이터베이스에 저장
   - 뷰 이름 앞에는 v_ 나 view_ 같은 접두사를 붙여서 다른 테이블과 쉽게 구분할 수 있도록 하는 것이 실무에서 흔히 사용하는 좋은 습관
```sql
DROP VIEW IF EXISTS v_category_order_status; -- 만약 뷰가 이미 존재한다면 제거

CREATE VIEW v_category_order_status
AS
SELECT
   p.category,
   COUNT(*) AS total_orders,
   SUM(CASE WHEN o.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed_count,
   SUM(CASE WHEN o.status = 'SHIPPED' THEN 1 ELSE 0 END) AS shipped_count,
   SUM(CASE WHEN o.status = 'PENDING' THEN 1 ELSE 0 END) AS pending_count
FROM
   orders o
JOIN
   products p ON o.product_id = p.product_id
GROUP BY
   p.category;
```
   - 이 쿼리를 실행하고 나면, 우리 데이터베이스에 v_category_order_status 라는 새로운 객체가 생성
   - 💡 이 뷰는 실제 데이터를 담고 있는 것이 아니라, 위 SELECT 쿼리문 자체를 품고 있는 것

2. 뷰 조회하기: SELECT
   - 뷰를 그냥 하나의 테이블이라고 생각하고 SELECT 문을 사용하면 됨
```sql
SELECT *
FROM v_category_order_status;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5beba085-ce02-4df5-b114-b6b8615bdc76">
</div>

  - 간단한 쿼리 한 줄로 복잡한 쿼리를 실행했을 때와 완벽하게 동일한 결과를 얻을 수 있음
  - 뷰 역시 테이블처럼 다룰 수 있으므로, WHERE 절이나 ORDER BY 를 사용해서 뷰의 결과를 필터링하거나 정렬할 수 있음
```sql
SELECT *
FROM v_category_order_status
WHERE category = '전자기기';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a24eb7e5-8d48-4efb-a5d0-8ada600f55cd">
</div>

3. ALTER VIEW : 뷰 수정하기
  - 뷰를 사용하다 보면, 저장된 쿼리의 내용을 수정해야 할 일이 발생
  - 예를 들어, 마케팅팀에서 "카테고리별 주문 건수뿐만 아니라, 총 매출액도 함께 보고 싶다는" 추가 요청을 했다고 가정
  - 이때 사용하는 명령어가 ALTER VIEW : 기존 뷰의 정의를 새로운 SELECT 쿼리로 덮어씀
```sql
ALTER VIEW v_category_order_status
AS
SELECT
   p.category,
   SUM(p.price * o.quantity) AS total_sales, -- 매출액 컬럼 추가!
   COUNT(*) AS total_orders,
   SUM(CASE WHEN o.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed_count,
   SUM(CASE WHEN o.status = 'SHIPPED' THEN 1 ELSE 0 END) AS shipped_count,
   SUM(CASE WHEN o.status = 'PENDING' THEN 1 ELSE 0 END) AS pending_count
FROM
   orders o
JOIN
   products p ON o.product_id = p.product_id
GROUP BY
   p.category;
```
  - 뷰의 정의를 수정한 뒤, 다시 한번 뷰를 조회
```sql
SELECT *
FROM v_category_order_status;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/257ea121-30c5-4190-830d-0adde11d888c">
</div>

   - total_sales 라는 새로운 컬럼이 성공적으로 추가된 것을 확인할 수 있음
   - 뷰를 사용하는 모든 팀은 이제 별도의 수정 없이 이 새로운 보고서를 즉시 받아볼 수 있게 됨

3. DROP VIEW : 뷰 삭제하기
   - 더 이상 사용하지 않는 뷰는 DROP VIEW 명령어로 간단하게 삭제할 수 있음
```sql
DROP VIEW v_category_order_status;
```
   - 여기서 가장 중요한 사실은, 뷰를 삭제해도 원본 테이블인 orders와 products의 데이터는 단 하나도 손상되거나 변경되지 않는다는 점
   - 그저 쿼리를 저장해 둔 편리한 '도구' 하나가 사라지는 것뿐임
