-----
### 문제와 풀이
-----
1. 문제 1 : 주문별 상품 정보 조회
   - INNER JOIN을 사용하여 orders 테이블과 products 테이블을 연결
   - 모든 주문에 대해 주문 ID, 상품명, 주문 수량 이 포함된 목록을 조회하는 SQL을 작성하고 order_id으로 오름차순 정렬
   - 가독성을 위해 테이블 별칭을 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/c18e4e22-0c87-4e53-ba09-5c99eaae5146">
</div>

   - 정답
```sql
SELECT
     o.order_id,
     p.name,
     o.quantity
FROM
     orders o
JOIN
     products p ON o.product_id = p.product_id
ORDER BY
     o.order_id;
```

2. 문제 2 : 3개 테이블 조인하기
  - orders, users, products 세 개의 테이블을 모두 조인
  - SHIPPED(배송) 상태인 주문에 대해 주문 ID, 고객 이름, 상품명, 주문 날짜를 조회하는 SQL을 작성
<div align="center">
<img src="https://github.com/user-attachments/assets/2d09f258-af64-48db-9cd6-6de6e3319403">
</div>

   - 정답
```sql
SELECT
     o.order_id,
     u.name AS user_name,
     p.name AS product_name,
     o.order_date
FROM
     orders o
JOIN
     users u ON o.user_id = u.user_id
JOIN
     products p ON o.product_id = p.product_id
WHERE
     o.status = 'SHIPPED';
```

3. 문제3 : 고객별 총 구매액 계산
   - INNER JOIN과 집계 함수를 함께 사용해서, 각 고객이 지금까지 주문한 총 구매액을 계산
   - 결과는 고객 이름(user_name)과 총 구매액(total_purchase_amount)으로 구성
   - 총 구매액이 높은 순서대로 정렬 (총 구매액 = 주문수량 * 상품가격)
<div align="center">
<img src="https://github.com/user-attachments/assets/c6ab6fc8-42ed-4554-bbdd-7cac1dad906f">
</div>

   - 정답
```sql
SELECT
     u.name AS user_name,
     SUM(o.quantity * p.price) AS total_purchase_amount
FROM
     orders o
JOIN
     users u ON o.user_id = u.user_id
JOIN
     products p ON o.product_id = p.product_id
GROUP BY
     u.name
ORDER BY
     total_purchase_amount DESC;
```
