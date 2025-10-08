-----
### 문제와 풀이 1
-----
1. 문제 1 : 특정 카테고리의 미판매 상품 찾기
   - products 테이블과 orders 테이블을 LEFT JOIN 하여, '전자기기' 카테고리에 속하지만 단 한 번도 판매되지 않은 상품의 이름과 가격을 조회하는 SQL을 작성
<div align="center">
<img src="https://github.com/user-attachments/assets/18555a1b-7d7e-4f90-bb1f-a79cf31aac4f">
</div>

   - 정답
```sql
SELECT
   p.name,
   p.price
FROM
   products p
LEFT JOIN
   orders o ON p.product_id = o.product_id
WHERE
   p.category = '전자기기' AND o.order_id IS NULL;
```
   - products를 기준 테이블로 삼아 LEFT JOIN을 수행
   - WHERE 절을 사용하여 category가 '전자기기'인 상품만 필터링하고, 그중에서 orders 테이블과 연결고리가 없는 (o.order_id IS NULL) 상품, 즉 판매되지 않은 상품을 찾음

2. 💡 문제 2 : 고객별 주문 횟수 구하기 (주문 없는 고객 포함)
   - 모든 고객의 이름과 각 고객이 주문한 총 횟수를 조회하는 SQL을 작성
   - 주문을 한 번도 하지 않은 고객은 주문 횟수가 0으로 표시
   - 결과는 고객 이름으로 오름차순 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/e24df91a-f8ed-4ea3-9eca-13e93e8ae6a9">
</div>

   - 힌트 : 주문이 없는 고객의 경우 o.order_id 가 NULL 이므로 COUNT 결과가 0이 되어야 함

```sql
```sql
SELECT
   u.name,
   COUNT(o.order_id) AS order_count
FROM
   users u
LEFT JOIN
   orders o ON u.user_id = o.user_id
GROUP BY
   u.user_id, u.name
ORDER BY
   u.name;
```
   - 모든 고객을 결과에 포함해야 하므로 users 테이블을 기준으로 LEFT JOIN 사용
   - COUNT(o.order_id)를 사용하면, 주문이 없는 고객의 경우 o.order_id가 NULL 이므로 COUNT 결과가 0
   - GROUP BY를 사용하여 고객별로 주문 횟수를 집계
     + 💡 u.user_id 를 기준으로 그룹화하는 것이 더 정확하지만, 여기서는 이름도 함께 그룹화하여 SELECT 절에 이름을 표시할 수 있도록 함

3. 문제 3 : RIGHT JOIN으로 주문 없는 고객 찾기
   - RIGHT JOIN을 사용하여, 가입은 했지만 주문 기록이 없는 고객의 이름과 이메일을 찾는 SQL을 작성
<div align="center">
<img src="https://github.com/user-attachments/assets/0c3c87f9-bc64-4857-baae-04f15e5f517f">
</div>

   - 정답
```sql
SELECT
   u.name,
   u.email
FROM
   orders o
RIGHT JOIN
   users u ON o.user_id = u.user_id
WHERE
   o.order_id IS NULL;
```
   - RIGHT JOIN을 사용하여 users 테이블을 기준 테이블로 만듬 : RIGHT JOIN 구문에서 오른쪽에 users 테이블을 위치시킴
   - 이렇게 하면 users 테이블의 모든 데이터가 결과에 포함되고, 주문 기록이 없는 고객의 orders 관련 컬럼은 NULL
   - WHERE o.order_id IS NULL 조건으로 주문 기록이 없는 고객만 필터링

4. 문제 4 : 고객별 주문 상품 목록 조회하기
   - 모든 고객의 이름과 그 고객이 주문한 상품의 이름을 조회하는 SQL을 작
   - 한 고객이 여러 상품을 주문했다면 모든 상품명이 나와야 하며, 주문 기록이 없는 고객의 상품명은 NULL 로 표시되어야 함
   - 결과는 고객 이름(user_name), 상품 이름 순(product_name)으로 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/99236a95-c2e8-4bcb-a0e9-b7cd26ab3a97">
</div>

```sql
SELECT
   u.name AS user_name,
   p.name AS product_name
FROM
   users u
LEFT JOIN
   orders o ON u.user_id = o.user_id
LEFT JOIN
   products p ON o.product_id = p.product_id
ORDER BY
   user_name, product_name;
```
   - users 테이블을 기준으로 orders 테이블에 LEFT JOIN하여 모든 고객을 포함시킴
   - 그 결과에 다시 한번 products 테이블을 LEFT JOIN하여 주문된 상품의 이름을 가져옴
   - 연속으로 LEFT JOIN을 사용함으로써, users(고객)를 최종 기준으로 삼고, 중간에 주문(orders)이 없더라도 고객 정보는 유지되며 product_name은 NULL로 표시

-----
### 문제와 풀이 2
-----
1. 문제 1 : 특정 상사의 부하 직원 찾기
   - employees 테이블을 셀프 조인(SELF JOIN)하여 '최과장'의 직속 부하 직원들의 이름과 직원 ID를 모두 조회하는 SQL 쿼리를 작성
<div align="center">
<img src="https://github.com/user-attachments/assets/c602ba25-070b-4a0f-b86c-5bfbe31d98d1">
</div>

```sql
SELECT
	  m.employee_id,
    m.name,
    m.manager_id,
    e.name AS manager_name
FROM
    employees m
JOIN
    employees e
ON
    m.employee_id = e.manager_id
WHERE
    m.name = '최과장';
```
   - m은 상사(manager) 테이블 별칭, e는 직원(employee) 테이블 별칭

2. 문제 2 : 모든 상품 옵션 조합에 재질 추가하기
   - sizes와 colors 테이블을 CROSS JOIN 하여 모든 색상과 사이즈 조합을 만들었던 것을 응용
   - '면(Cotton)'과 '실크(Silk)'라는 두 가지 재질 옵션을 가진 materials 테이블을 새로 만들 것
   - 기존의 sizes, colors 테이블과 모두 CROSS JOIN 하여 '상품명-색상-사이즈-재질' 형태의 모든 조합을 조회하는 쿼리를 작성
      + materials 테이블을 생성하고 데이터를 삽입
      + sizes, colors, materials 세 테이블을 CROSS JOIN
      + CONCAT 함수를 사용하여 상품명-색상-사이즈-재질 형식의 product_full_name을 생성
<div align="center">
<img src="https://github.com/user-attachments/assets/445aad7a-2754-4271-b21e-8ec7193ecad4">
</div>

   - 총 24개 행 출력
```sql
-- 실습을 위한 임시 테이블 생성 및 데이터 삽입
-- (테이블이 이미 존재할 경우를 대비하여 DROP 구문 추가)
DROP TABLE IF EXISTS materials;

CREATE TABLE materials (
   material VARCHAR(20) PRIMARY KEY
);

INSERT INTO materials(material) VALUES ('Cotton'), ('Silk');
```
```sql
-- 세 테이블을 CROSS JOIN하여 모든 조합 조회
SELECT
   CONCAT('기본티셔츠-', c.color, '-', s.size, '-', m.material) AS
product_full_name,
   s.size,
   c.color,
   m.material
FROM
   sizes s
CROSS JOIN
   colors c
CROSS JOIN
   materials m
ORDER BY
   s.size, c.color, m.material;
```

3. 문제 3 : 특정 고객의 주문 내역 상세 조회하기
   - users, orders, products 세 개의 테이블을 조인하여 '네이트' 고객이 주문한 모든 상품의 이름(product_name), 주문 날짜(order_date), 주문 수량(quantity)을 조회하는 SQL 쿼리를 작성
   - 결과는 주문 날짜 최신순으로 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/d8c3bfa1-8d69-489e-a087-63986af3731a">
</div>

```sql
SELECT
   u.name AS customer_name,
   p.name AS product_name,
   o.order_date,
   o.quantity
FROM
   users u
JOIN
   orders o ON u.user_id = o.user_id
JOIN
   products p ON o.product_id = p.product_id
WHERE
   u.name = '네이트'
ORDER BY
   o.order_date DESC;
```

4. 문제 4 : 서울 지역 고객의 총 주문 금액 계산하기
   - users, orders, products 테이블을 조인하여 '서울'에 거주하는 고객별로 총 주문 금액(total_spent)을 계산하는 쿼리
   - 총 주문 금액은 각 주문의 가격(price) * 수량(quantity)의 합계
   - 결과는 총 주문 금액이 높은 순으로 정렬
<div align="center">
<img src="https://github.com/user-attachments/assets/43ccf88d-04ca-4209-ae07-09cb1d25a986">
</div>

```sql
```sql
SELECT
    u.name AS customer_name,
    SUM(p.price * o.quantity) AS total_spent
FROM
    users u
JOIN
    orders o ON u.user_id = o.user_id
JOIN
    products p ON o.product_id = p.product_id
WHERE
    u.address LIKE '서울%'
GROUP BY
    u.name
ORDER BY
    total_spent DESC;
```

