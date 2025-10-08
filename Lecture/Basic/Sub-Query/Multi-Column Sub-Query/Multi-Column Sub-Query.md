-----
### 다중 컬럼 서브쿼리
-----
1. 이전까지 우리는 서브쿼리가 하나의 컬럼만을 반환하는 경우만 다룸
   - 하지만 서브쿼리는 SELECT 문이므로, 당연히 여러 개의 컬럼을 반환할 수도 있음

2. 다중 컬럼 서브쿼리(Multi-Column Subquery)는 서브쿼리의 SELECT 절에 두 개 이상의 컬럼이 포함되는 경우
   - 이 기법은 메인쿼리의 WHERE 절에서 여러 컬럼을 동시에 비교해야 할 때 매우 유용

3. 우리 쇼핑몰의 고객 '네이트'(user_id =2)가 한 주문(order_id =3)이 있는데, 이 주문과 동일한 고객이면서 주문 처리 상태(status)도 같은 모든 주문을 검색
   - 이 문제의 핵심은 user_id와 status라는 두 개의 조건을 동시에 만족하는 주문을 찾는 것

4. 다중 컬럼 서브쿼리로 문제 해결하기
   - 이 문제를 해결하기 위해, 우리는 서브쿼리를 통해 order_id가 3인 주문의 user_id와 status를 한 번에 조회하고, 메인쿼리에서는 이 두 값을 동시에 비교할 것
   - 1단계 (서브쿼리) : 비교 기준이 될 고객 ID와 주문 상태 조회
      + 먼저 order_id 가 3인 주문의 고객 ID와 주문 상태를 조회하는 서브쿼리
```sql
SELECT user_id, status
FROM orders
WHERE order_id = 3;
```
   - order_id 는 PK 이므로 이 쿼리는 반드시 하나의 행만 반환
<div align="center">
<img src="https://github.com/user-attachments/assets/057e2d4b-8657-4ae6-b7c1-cdbdaa6ea054">
</div>

  - 서브쿼리는 (2, 'SHIPPED') 라는 단일 행, 다중 컬럼 결과를 반환

   - 2단계 (메인쿼리) : 다중 컬럼 비교
     + 이제 이 서브쿼리를 메인쿼리의 WHERE 절에 넣어서 두 개의 컬럼을 한 번에 비교
     + 이 때, 메인쿼리의 WHERE 절에도 비교할 컬럼들을 괄호로 묶어 (user_id, status) 와 같은 형태로 작성해주어야 함
```sql
SELECT order_id, user_id, status, order_date
FROM orders
WHERE (user_id, status) = (SELECT user_id, status
                           FROM orders
                           WHERE order_id = 3);
```
   - 이 쿼리는 다음과 같은 형태로 변환
```sql
SELECT order_id, user_id, status, order_date
FROM orders
WHERE (user_id, status) = (2, 'SHIPPED');
```
   - 쿼리 실행 흐름
     + 괄호 안의 서브쿼리가 먼저 실행되어 (2, 'SHIPPED') 라는 한 쌍의 값을 반환
     + 💡 메인쿼리는 WHERE (user_id, status) = (2, 'SHIPPED') 와 동일한 형태로 바뀜 (WHERE user_id = 2 AND status = 'SHIPPED' 와 논리적으로 같음)
     + 최종적으로 orders 테이블에서 user_id가 2이고 status 가 'SHIPPED'인 모든 주문을 찾아 반환
<div align="center">
<img src="https://github.com/user-attachments/assets/aff268c8-b178-4c00-96eb-7777994c7c88">
</div>

   - 결과를 보면 기준이 된 주문(order_id=3)을 포함하여, 고객('네이트', id=2)과 주문 상태('SHIPPED')가 모두 동일한 주문들이 성공적으로 조회된 것을 확인할 수 있음
   - 만약 여기서 기준이 된 주문 자신을 제외하고 싶다면, WHERE 절에 조건을 하나 더 추가하면 됨
```sql
SELECT order_id, user_id, status, order_date
FROM orders
WHERE (user_id, status) = (SELECT user_id, status
                           FROM orders
                           WHERE order_id = 3)
AND order_id != 3; -- 자기 자신은 제외
```
<div align="center">
<img src="https://github.com/user-attachments/assets/1869f78b-950a-4fad-b149-3e68bbbe6953">
</div>

5. 주의할 점
   - 다중 컬럼 서브쿼리를 = 연산자와 함께 사용할 때는, 단일 행 서브쿼리와 마찬가지로 서브쿼리의 결과가 반드시 하나의 행이어야 한다는 점을 잊지 말아야 함
   - 만약 서브쿼리가 두 개 이상의 행을 반환하면 데이터베이스는 어떤 행과 비교 해야 할지 알 수 없으므로 오류를 발생시킴

6. 다중 컬럼 서브쿼리와 IN 연산자
   - 다중 컬럼 서브쿼리 역시 서브쿼리의 결과가 여러 행일 수 있는데, 이때는 = 대신 IN 연산자를 사용해야 함
   - 요구사항 : 각 고객별로 가장 먼저 한 주문의 주문ID, 사용자ID, 사용자이름, 제품이름, 주문 날짜를 조회
     + 먼저 각 고객별로 가장 빠른 주문 날짜를 구하는 서브쿼리를 작성
```sql
SELECT user_id, MIN(order_date)
FROM orders
GROUP BY user_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/d83e5eaf-4cbf-438a-80b2-7a3e8d3bcde1">
</div>

   - 이 서브쿼리는 여러 고객의 첫 주문 정보를 담고 있으므로, 여러 행과 여러 열을 반환   
   - 이제 이 서브쿼리의 결과를 이용해 각 고객의 첫 주문 정보를 조회
     + 우선 단순하게 조회해보자.
     + 이때는 서브쿼리가 여러 개의 (user_id, order_date) 쌍을 반환하므로 = 연산자 대신 IN 을 사용
```sql
SELECT
   o.order_id,
   o.user_id,
   o.order_date
FROM orders o
WHERE (o.user_id, o.order_date) IN (SELECT user_id, MIN(order_date)
                                     FROM orders
                                     GROUP BY user_id);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/103ec97e-b9b0-4f0c-85ea-b36dc65747ba">
</div>

   - 필요한 주문ID, 사용자ID, 사용자이름, 제품이름, 주문 날짜를 모두 조회
```sql
SELECT
   o.order_id,
   o.user_id,
   u.name,
   p.name AS product_name,
   o.order_date
FROM orders o
JOIN users u ON o.user_id = u.user_id
JOIN products p ON o.product_id = p.product_id
WHERE (o.user_id, o.order_date) IN (SELECT user_id, MIN(order_date)
                                     FROM orders
                                     GROUP BY user_id);
```
   - 메인쿼리의 각 행에 대해 (user_id, order_date) 쌍이 서브쿼리가 반환한 여러 쌍 중 하나와 일치하는 경우에만 최종 결과에 포함
   - 이처럼 다중 컬럼 서브쿼리는 두 개 이상의 속성이 조합되어 특정 의미를 가질 때, 그 조합 자체를 조건으로 사용하여 데이터를 조회하는 간결하고 강력한 방법을 제공
