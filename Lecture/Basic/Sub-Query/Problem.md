-----
### 문제와 풀이
-----
1. 문제 1 : 가장 비싼 상품 조회하기
   - products 테이블에서 가격이 가장 비싼 상품의 product_id, name, price를 조회
   - WHERE 절에 스칼라 서브쿼리를 사용하여 문제를 해결
<div align="center">
<img src="https://github.com/user-attachments/assets/737bdf2b-4389-41a1-9a74-02d86ad78990">
</div>

```sql
SELECT
   product_id AS 상품ID,
   name AS 상품명,
   price AS 가격
FROM
   products
WHERE
   price = (SELECT MAX(price) FROM products);
```

2. 💡 문제 2 : 동일 상품 주문 정보 조회하기
   - order_id가 1인 주문과 동일한 상품을 주문한 다른 모든 주문의 order_id, user_id  order_date를 조회
   - 스칼라 서브쿼리를 활용
<div align="center">
<img src="https://github.com/user-attachments/assets/f7d7d4bb-6189-45a3-8a26-6016dbf78edf">
</div>

```sql
SELECT
    order_id AS 주문ID,
    user_id AS 고객ID,
    order_date AS 주문일시
FROM
    orders
WHERE
    product_id = (SELECT product_id
                  FROM orders
                  WHERE order_id = 1)
AND order_id != 1;
```

3. 💡 문제 3: 고객별 총 주문 횟수 조회하기
   - 각 고객(users)별로 총 몇 번의 주문을 했는지 '총주문횟수'를 이름과 함께 조회
   - 정렬은 user_id 오름차순
   - 한 번도 주문하지 않은 고객도 결과에 포함되어야 함
   - SELECT 절에 상관 서브쿼리를 사용하여 해결
<div align="center">
<img src="https://github.com/user-attachments/assets/36116bd0-9425-4cbc-8905-c3de6570dbb0">
</div>

```sql
SELECT
    u.name AS 고객명,
    (SELECT COUNT(*) FROM orders WHERE o.user_id = u.user_id) AS 총주문횟수
FROM
    users u
ORDER BY
    u.user_id;
```
