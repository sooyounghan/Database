-----
### 문제와 풀이
-----
1. 문제 1 : 상품 카테고리 영문으로 표시하기
   - products 테이블에서 상품의 category 컬럼 값을 영문으로 변경하여 조회
   - '전자기기'는 'Electronics', '도서'는 'Books', '패션'은 'Fashion'으로 표시하고, category_english 라는 별칭을 사용
   - 상품 이름, 원래 카테고리, 그리고 영문 카테고리를 함께 출력
<div align="center">
<img src="https://github.com/user-attachments/assets/f8ab40c0-791e-4125-9558-574c6f7e4a0f">
</div>

```sql
SELECT
   name,
   category,
   CASE category
   WHEN '전자기기' THEN 'Electronics'
   WHEN '도서' THEN 'Books'
   WHEN '패션' THEN 'Fashion'
   ELSE 'Etc'
   END AS category_english
FROM
   products;
```

2. 문제 2 : 주문 수량에 따른 분류 및 정렬
   - orders 테이블의 주문들을 수량(quantity)에 따라 분류
   - 수량이 2개 이상이면 '다량 주문', 1개이면 '단일 주문'으로 표시하는 order_type 컬럼
   - 주문 ID, 수량, 그리고 이 새로운 order_type 컬럼을 조회하되, '다량 주문'이 '단일 주문'보다 먼저 나오도록 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/d57e1dd4-14a0-43ad-9be3-198d1685e35e">
</div>

```sql
SELECT
     order_id,
     quantity,
     CASE
         WHEN quantity >= 2 THEN '다량 주문'
         ELSE '단일 주문'
     END AS order_type
FROM
     orders
ORDER BY
     CASE
         WHEN quantity >= 2 THEN 1
         ELSE 2
     END ASC;
```

3. 문제 3 : 재고 수준별 상품 수 집계하기
   - products 테이블의 상품들을 재고 수량( stock_quantity )에 따라 그룹화하여 각 그룹에 속한 상품의 수 확인
      + 재고가 50개 이상이면 '재고 충분'
      + 20개 이상 50개 미만이면 '재고 보통'
      + 20개 미만이면 '재고 부족'
   - stock_level과 product_count라는 컬럼명으로 결과를 출력
<div align="center">
<img src="https://github.com/user-attachments/assets/782aeee9-1977-417e-a75e-c5d4bd7b1d11">
</div>

```sql
SELECT
     CASE
         WHEN stock_quantity >= 50 THEN '재고 충분'
         WHEN stock_quantity >= 20 THEN '재고 보통'
         ELSE '재고 부족'
     END AS stock_level,
     COUNT(*) AS product_count
FROM
     products
GROUP BY
     stock_level;
```

4. 💡 문제 4 : 사용자별 카테고리 주문 건수 피벗 테이블 만들기
   - 각 사용자(users)가 카테고리별로 몇 건의 주문을 했는지 피벗 테이블 형태로 조회
   - 사용자 이름, 총 주문 건수, '전자기기' 주문 건수, '도서' 주문 건수를 각각 user_name, total_orders, electronics_orders, book_orders 컬럼으로 출력
   - 한 번도 주문하지 않은 고객도 결과에 포함
<div align="center">
<img src="https://github.com/user-attachments/assets/cd87f4b9-3e6e-445d-a24e-f87fda082de9">
</div>

```sql
SELECT
     u.name AS user_name,
     COUNT(o.order_id) AS total_orders,
     SUM(CASE WHEN p.category = '전자기기' THEN 1 ELSE 0 END) AS electronics_orders,
     SUM(CASE WHEN p.category = '도서' THEN 1 ELSE 0 END) AS book_orders,
     SUM(CASE WHEN p.category = '패션' THEN 1 ELSE 0 END) AS fashion_orders
FROM
     users u
LEFT JOIN
     orders o ON u.user_id = o.user_id
LEFT JOIN
     products p ON o.product_id = p.product_id
GROUP BY
     u.name;
```
