-----
### CASE 문 - 조건부 집계
-----
1. CASE 문이 집계 함수의 '바깥'에서 그룹의 기준을 만드는 역할을 했음
2. CASE 문은 또한, 집계 함수(SUM, COUNT 등)의 '안'으로 들어가는, 훨씬 더 강력한 활용 가능 : 이 기법을 조건부 집계(Conditional Aggregation)
3. 상황 : 엑셀의 '피벗 테이블' 기능과 유사한 보고서를 SQL로 직접 만들어보는 것
   - 하나의 쿼리로, 전체 주문 건수와 함께 '결제 완료(COMPLETED)', '배송(SHIPPED)', '주문 대기(PENDING)' 상태의 주문이 각각 몇 건인지 별도의 컬럼으로 출력
<div align="center">
<img src="https://github.com/user-attachments/assets/24f5761b-13ca-4a6d-ba1d-cabc37a72d27">
</div>

   - 피벗 테이블(Pivot Table) : 데이터를 다양한 관점에서 회전(pivot)시켜 분석할 수 있는 기능 때문에 붙여진 이름

4. 단계별 접근
   - 먼저 주문 테이블의 데이터를 확인
```sql
SELECT order_id, status FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/da23baa5-48aa-464b-9b0a-f30d08d43401">
</div>

   - 이 테이블을 각 주문의 상태별로 그룹핑
```sql
SELECT status, COUNT(*)
FROM orders
GROUP BY status;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/00bd8630-a2d9-4975-90c7-e74a10771959">
</div>

   - '결제 완료(COMPLETED)', '배송(SHIPPED)', '주문 대기(PENDING)' 상태의 주문이 각각 몇 건인지 확인
   - 그런데 전체 GROUP BY를 사용했기 때문에 각 그룹별 통계만 구할 수 있음
   - 전체 합 구하기
     + 전체 합을 구하려면 GROUP BY 절을 사용하지 않고 COUNT(*)와 같은 집계 함수를 사용하면 됨
```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/ccdd4506-8362-43d1-beda-5300db4e2d68">
</div>

   - 이제 둘을 UNION으로 합치는 방법도 있고, 서브 쿼리를 사용하는 방법도 있음

5. UNION으로 합치는 방법
   - UNION을 사용하면 여러 SELECT 문의 결과를 하나로 합칠 수 있음
   - 이때 각 SELECT 문의 컬럼 수가 같아야 하며, 컬럼의 데이터 타입도 호환되어야 함
```sql
SELECT 'Total' AS category, COUNT(*) AS count
FROM orders

UNION ALL

SELECT status AS category, COUNT(*) AS count
FROM orders
GROUP BY status;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/592af1c3-c5aa-40d7-9ec7-902c58b132b5">
</div>

   - 이 방법은 각 상태별 통계와 전체 합계를 하나의 결과 집합으로 보여주지만, '각 상태별 카운트가 별도의 컬럼으로 나오는' 보고서 형식과는 차이가 있음
   - 이 모양을 옆으로 돌려서 피벗 테이블의 형태가 되어야 함

6. 서브 쿼리를 사용하는 방법
   - 각 상태별 카운트를 서브 쿼리로 미리 구한 다음, 메인 쿼리에서 이 값들을 조인하거나 활용하는 방법
```sql
SELECT
   (SELECT COUNT(*) FROM orders) AS total_orders,
   (SELECT COUNT(*) FROM orders WHERE status = 'COMPLETED') AS completed_count,
   (SELECT COUNT(*) FROM orders WHERE status = 'SHIPPED') AS shipped_count,
   (SELECT COUNT(*) FROM orders WHERE status = 'PENDING') AS pending;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c70a9ff9-c1f5-4ea2-bb71-36be1dba6801">
</div>

   - 이 방법은 우리가 목표로 하는 보고서 형식과 같은 결과를 보여줌
   - 각 카운트가 별도의 컬럼으로 잘 분리되어 있음
   - 이 방법은 우리가 원하는 결과를 주지만, 각 값을 얻기 위해 orders 테이블을 총 4번이나 읽어오는(FROM orders) 심각한 성능 문제를 가지고 있음 : 데이터가 많아질수록 매우 비효율적
   - 바로 이럴 때 CASE 문을 집계 함수 안에 넣는 조건부 집계 기법이 필요 : 이 방법은 단일 쿼리 내에서 효율적으로 조건별 집계를 수행할 수 있게 해줌

-----
### 💡 CASE를 품은 집계 함수
-----
1. 집계 함수의 인자로 CASE 문을 넣어 '조건에 맞을 때만 세거나 더하게' 만드는 것 - 두 가지 대표적인 패턴
2. 패턴 1 : COUNT(CASE ...)
   - COUNT 함수는 NULL이 아닌 모든 값을 센다는 특징을 이용
```sql
COUNT(CASE WHEN status = 'COMPLETED' THEN 1 END)
```
   - status가 'COMPLETED'이면 CASE문은 숫자 1을 반환
   - 그 외의 경우(ELSE가 없으므로), CASE 문은 NULL을 반환
   - 결과적으로 COUNT 함수는 status가 'COMPLETED'인 행의 개수만 세게 됨

3. 패턴 2 : SUM(CASE ...)
   - SUM 함수는 숫자들의 합계를 구함
```sql
SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END)
```
   - status 가 'COMPLETED'이면 CASE 문은 1을, 그 외에는 0을 반환
   - SUM 함수가 이 1과 0들을 모두 더하면, 그 합계는 결국 'COMPLETED' 상태인 주문의 총 개수
   - 두 패턴 모두 동일한 결과를 내며, 실무에서 널리 쓰임

4. 실습 1 : 전체 주문 상태 요약하기
   - 먼저 orders 테이블 전체를 대상으로, 각 상태별 주문 건수를 집계
```sql
SELECT
   COUNT(*) AS total_orders,
   SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed_count,
   SUM(CASE WHEN status = 'SHIPPED' THEN 1 ELSE 0 END) AS shipped_count,
   SUM(CASE WHEN status = 'PENDING' THEN 1 ELSE 0 END) AS pending_count
FROM
   orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e3eee023-ded2-45d1-b707-7cb7c58824af">
</div>

   - 하나의 쿼리로 행으로 흩어져 있던 status 정보를 열로 변환하여, 한눈에 파악하기 쉬운 피벗(Pivot) 보고서 생성

5. 실습 2 : GROUP BY 와 함께 사용하기 (피벗 테이블)
   - 상품 카테고리별로, 상태별 주문 건수를 집계하는 경우
   - 이를 위해서는 orders 테이블과 products 테이블을 JOIN하여 category 정보를 가져온 뒤, p.category를 기준으로 GROUP BY
```sql
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
<div align="center">
<img src="https://github.com/user-attachments/assets/3e98b5f6-4731-4d22-8fc6-f248faffd8b8">
</div>

   - 이제 우리는 카테고리별로 주문 현황을 훨씬 더 상세하고 입체적으로 분석할 수 있게 됨
   - '전자기기'는 주문은 많지만 배송 상태이거나 대기중인 건이 많은 반면, '도서'는 주문 건수는 적지만 모두 처리가 완료되었다는 인사이트를 단번에 얻을 수 있음

6. 이처럼 CASE 문을 집계 함수 내부에 사용하는 조건부 집계 기법은, 데이터를 단순히 요약하는 것을 넘어 원하는 형태로 재구조화하고 분석하는 데 필요한 도구
