-----
### 실습 2: LEFT JOIN 으로 '단 한 번도 팔리지 않은 상품' 찾기
-----
1. "단 한 번도 팔리지 않은 상품은 무엇인가?" : 이번 분석의 기준은 '상품'
   - products 테이블의 데이터는 모두 결과에 나와야 함
   - 따라서 products 테이블이 기준이 되어야 함
   - products 테이블 주요 정보
```sql
SELECT
   product_id,
   name,
   price
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/077d50e2-309c-4b88-8261-f1bb36c1eaa0">
</div>

   - orders 테이블 주요 정보
```sql
SELECT
   product_id,
   order_id
FROM orders
order by product_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/4ede310e-8f1c-466d-9d17-286b03ccf37c">
</div>

   - product_id로 정렬해서 조인을 더 이해하기 쉽게 확인

2. 1단계 : products를 기준으로 orders 테이블 LEFT JOIN 하기
   - products 테이블을 기준으로 LEFT JOIN을 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/fb541b7a-1267-4eac-82fd-799d18ebdf5f">
</div>

```sql
SELECT
   p.product_id,
   p.name,
   p.price,
   o.product_id,
   o.order_id
FROM products p
LEFT JOIN orders o ON p.product_id = o.product_id
```
   - 조인 진행
<div align="center">
<img src="https://github.com/user-attachments/assets/d8b88c07-ec72-455c-8c2b-d7a01f4c5614">
</div>

   - 고급 가죽 지갑, 스마트 워치는 주문 내역에 없음
<div align="center">
<img src="https://github.com/user-attachments/assets/b8a63d93-579d-497a-873c-465e678d9f6d">
</div>

   - products 테이블을 기준으로 LEFT OUTER JOIN 했으므로 products의 제품은 조인 결과에 모두 포함
   - 따라서 주문 내역이 없는 고급 가죽 지갑, 스마트 워치도 조인 결과에 포함 : 이 경우 조인 결과의 주문 관련 필드 값은 NULL

3. 2단계 : NULL 인 데이터만 필터링하기
   - 주문 정보가 NULL 인 상품이 바로 '단 한 번도 팔리지 않은 상품'
   - WHERE 절을 사용해서 이들만 찾아보기
```sql
SELECT
     p.product_id,
     p.name,
     p.price,
     o.product_id,
     o.order_id
FROM products AS p
LEFT JOIN orders AS o ON p.product_id = o.product_id
WHERE o.order_id IS NULL;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8ae31be2-9347-49fa-8173-0acccaf75e77">
</div>

   - 고급 가죽 지갑, 스마트 워치가 단 한 번도 팔리지 않은 상품인 것을 알 수 있음

-----
### 실습 3 : RIGHT JOIN 으로 '단 한 번도 팔리지 않은 상품' 찾기
-----
1. RIGHT JOIN 은 왼쪽이 아니라 오른쪽이 기준 : 따라서 오른쪽 테이블의 데이터가 조인 결과로 모두 나와야 함
2. 여기서 왼쪽은 FROM 절의 테이블이고, 오른쪽은 JOIN 절의 테이블을 말함
   - 앞서 진행했던 예시에서 테이블의 위치만 변경하면 됨
   - 그리고 이번에도 동일하게 '단 한 번도 팔리지 않은 상품'을 찾으면 됨

3. 1단계 : products를 기준으로 orders 테이블 RIGHT JOIN 하기
   - products 테이블을 기준으로 RIGHT JOIN 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/65798785-288b-4225-bdb9-ca65c8e60b9a">
</div>

```sql
SELECT
     p.product_id,
     p.name,
     p.price,
     o.product_id,
     o.order_id
FROM orders AS o
RIGHT JOIN products AS p ON o.product_id = p.product_id
```
   - 여기서는 orders가 FROM 다음에 나오고 products가 JOIN 다음에 나옴 : 둘의 위치를 바꿈

<div align="center">
<img src="https://github.com/user-attachments/assets/d8c7becc-aed7-41a1-bfe5-dab4512fc249">
</div>

   - orders 테이블이 왼쪽, products 테이블이 오른쪽
   - RIGHT JOIN은 오른쪽이 기준
   - products.product_id → orders.product_id로 조인
<div align="center">
<img src="https://github.com/user-attachments/assets/1316ee3f-8c5c-41be-be97-025026c4b2ef">
</div>

   - RIGHT OUTER JOIN을 사용했으므로 오른쪽에 있는 products 테이블이 기준 테이블이 되고, products 테이블은 조인 결과에 모두 포함
   - 따라서 주문 내역이 없는 고급 가죽 지갑, 스마트 워치도 조인 결과에 포함
   - 이 경우 조인 결과의 주문 관련 필드 값은 NULL

4. 2단계 : NULL 인 데이터만 필터링하기
```sql
SELECT
   p.product_id,
   p.name,
   p.price,
   o.product_id,
   o.order_id
FROM orders AS o
RIGHT JOIN products AS p ON o.product_id = p.product_id
WHERE o.order_id IS NULL;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/9386da74-a09e-4b2d-a17b-51ec7a8ef432">
</div>

   - 고급 가죽 지갑, 스마트 워치가 단 한 번도 팔리지 않은 상품인 것을 알 수 있음

-----
### LEFT JOIN과 RIGHT JOIN 선택
-----
1. LEFT JOIN과 RIGHT JOIN은 서로 위치를 바꾸어 사용할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/fa709a9e-a41b-402c-b6fa-4cf081c3ebb9">
</div>

```sql
SELECT
   p.product_id,
   p.name,
   p.price,
   o.product_id,
   o.order_id
FROM products AS p
LEFT JOIN orders AS o ON p.product_id = o.product_id
WHERE o.order_id IS NULL;
```
   - products는 왼쪽, orders는 오른쪽
   - LEFT JOIN을 사용하면 products를 기준 테이블로 정할 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/c92b78bb-fc9c-4912-98e3-427af62390b1">
</div>

```sql
SELECT
     p.product_id,
     p.name,
     p.price,
     o.product_id,
     o.order_id
FROM orders AS o
RIGHT JOIN products AS p ON o.product_id = p.product_id
WHERE o.order_id IS NULL;
```
   - orders는 왼쪽, products는 오른쪽
   - RIGHT JOIN을 사용하면 products를 기준 테이블로 정할 수 있음

2. 두 SQL의 실행 결과는 같음 : 테이블을 어떤 위치에 두든 LEFT, RIGHT로 기준 테이블을 정하면 됨

3. 실무에서는 LEFT JOIN이 RIGHT JOIN 보다 훨씬 더 많이 사용
   - 💡 보통 분석의 기준이 되는 테이블을 FROM 절에 먼저 쓰고, 필요한 정보를 담은 다른 테이블들을 LEFT JOIN 으로 하나씩 붙여나가는 방식으로 쿼리를 작성하는 것이 더 직관적이기 때문임
   - RIGHT JOIN은 테이블의 순서를 바꾸면 언제나 LEFT JOIN으로 동일하게 표현할 수 있음

4. 정리
   - LEFT JOIN과 RIGHT JOIN은 조인할 때 기준 테이블의 위치를 정함
   - LEFT JOIN은 기준 테이블이 FROM 절에 먼저 나오므로 더 읽기 편함 : 따라서 기준 테이블을 FROM 절에 먼저 쓰는 습관을 들이는 것이 좋음
   - FROM과 JOIN 절에 있는 테이블의 위치를 바꾸면 LEFT JOIN을 RIGHT JOIN으로 변경할 수 있음
   - 반대로, FROM과 JOIN 절에 있는 테이블의 위치를 바꾸면 RIGHT JOIN을 LEFT JOIN으로 변경할 수 있음
   - 이러한 이유로 실무에서는 대부분 LEFT JOIN을 사용하며, RIGHT JOIN은 잘 사용하지 않음
