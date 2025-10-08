-----
### 조인 종합 실습
-----
1. 2025년 6월에 '서울'에 거주하는 고객이 주문한 모든 내역에 대해, 고객 이름, 고객 이메일, 주문 날짜, 주문한 상품명, 상품 가격, 주문 수량 을 포함하는 상세 보고서를 최신 주문 순으로 작성
2. 문제 분석 및 해결 전략
   - 필요한 정보
      + 고객 이름, 고객 이메일 → users 테이블
      + 주문 날짜, 주문 수량 → orders 테이블
      + 주문한 상품명, 상품 가격 → products 테이블
      + 주문 내역에 대한 것이므로 주문 테이블을 기준으로 시작하는 것이 좋음
   - 필요한 테이블 : 위 정보를 모두 얻으려면 users, orders, products 세 개의 테이블이 모두 필요
   - 필터링 조건
      + 조건 1 : 2025년 6월에 이루어진 주문 (orders 테이블)
      + 조건 2 : '서울'에 거주하는 고객의 주문 (users 테이블)
   - 정렬 순서 : 최신 주문 순서 (orders 테이블의 order_date 기준 내림차순)

   - users, orders, products 세 테이블을 모두 조인한 후, WHERE 절로 두 가지 조건을 필터링하고, 최종 결과를 ORDER BY로 정렬하면 됨

3. 1단계 : orders와 users 조인하기
  - 가장 먼저 주문과 고객을 연결
  - 주문 기록이 있는 고객만 필요하므로 INNER JOIN이 적합
```sql
SELECT
   u.user_id, u.name, u.email, u.address, -- 고객
   o.order_id, o.order_date, o.quantity -- 주문
FROM
   orders o
JOIN
   users u ON o.user_id = u.user_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5a6a46e7-35ab-4281-9a97-f7b3709b7649">
</div>

   - 고객 테이블과 주문 테이블을 조인을 사용해서 연결

4. 2단계 : products 테이블 추가로 조인하기
   - 위에서 조인된 결과에 이제 상품 정보를 연결 : orders 테이블의 product_id와 products 테이블의 product_id를 연결고리로 사용
```sql
SELECT
   u.user_id, u.name, u.email, u.address, -- 고객
   o.order_id, o.order_date, o.quantity, -- 주문
   p.product_id, p.name, p.price -- 상품
FROM
   orders o
JOIN
   users u
ON
   o.user_id = u.user_id
JOIN
   products p
ON
   o.product_id = p.product_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/bfb3e104-3398-4192-b33e-db7977d4bf1b">
</div>

   - 고객 테이블과 주문 테이블을 그리고 상품 테이블을 조인을 사용해서 연결
   - 이렇게 JOIN 구문은 체인처럼 계속 이어서 작성할 수 있음

5. 3단계 : WHERE 절로 조건 필터링하기
   - 이제 두 가지 필터링 조건을 WHERE 절에 추가 (여러 조건은 AND 로 연결)
   - 거주지 조건 : 주소가 '서울'로 시작하는 모든 고객을 찾기 위해 LIKE '서울%' 을 사용
   - 날짜 조건 : 6월 날짜의 범위를 지정
```sql
SELECT
   u.user_id, u.name, u.email, u.address, -- 고객
   o.order_id, o.order_date, o.quantity, -- 주문
   p.product_id, p.name, p.price -- 상품
FROM
   orders o
JOIN
   users u ON o.user_id = u.user_id
JOIN
   products p ON o.product_id = p.product_id
WHERE
   u.address LIKE '서울%'
AND
   o.order_date >= '2025-06-01' AND o.order_date < '2025-07-01';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/b8029e95-6e38-4ef3-9ff1-576e19443e0a">
</div>

6. 4단계 : 최종 보고서 양식에 맞게 컬럼 선택 및 정렬하기
   - 마지막으로 문제에서 요구한 컬럼들만 SELECT 절에 명시하고, ORDER BY를 사용해 최신순으로 정렬하면 모든 요구 사항이 반영된 최종 쿼리가 완성
```sql
SELECT
   u.name AS customer_name,
   u.email,
   o.order_date,
   p.name AS product_name,
   p.price,
   o.quantity
FROM
   orders o
JOIN
   users u
ON
   o.user_id = u.user_id
JOIN
   products p
ON
   o.product_id = p.product_id
WHERE
   u.address LIKE '서울%'
AND
   o.order_date >= '2025-06-01' AND o.order_date < '2025-07-01'
ORDER BY
   o.order_date DESC;
```
   - 컬럼 별칭(AS)을 사용해서 보고서의 헤더를 customer_name, product_name 처럼 더 명확하게 만듬
<div align="center">
<img src="https://github.com/user-attachments/assets/7805b39c-93ba-4580-a530-b74abdbc35ad">
</div>

