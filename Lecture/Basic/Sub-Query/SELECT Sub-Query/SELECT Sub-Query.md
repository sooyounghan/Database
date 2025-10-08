-----
### SELECT 서브쿼리
-----
1. 서브쿼리를 SELECT 절 안으로 가져온다면, 서브쿼리는 더 이상 필터가 아닌, 그 자체가 하나의 '컬럼'처럼 동작
   - 참고로 SELECT 절에서는 단일 값(하나의 행, 하나의 컬럼)을 반환하는 스칼라 서브쿼리를 사용해야 gka

2. 비상관 서브쿼리
   - 모든 상품 목록을 조회하는데, 각 상품의 가격과 함께 전체 상품의 평균 가격을 모든 행에 함께 표시해서 개별 상품 가격이 평균과 얼마나 차이 나는지 비교해보고 싶을 경우
   - 이 경우, '전체 상품의 평균 가격'은 어떤 특정 상품 행에 종속되는 값이 아니라, 모든 상품에 대해 동일하게 적용되는 고정된 값 : 이런 경우에 간단한 스칼라 서브쿼리를 사용할 수 있음
   - 전체 상품의 평균 가격을 구하는 쿼리
```sql
SELECT AVG(price)
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e7322fa2-402d-461f-bbb3-dae39e632d01">
</div>

   - SELECT 절 안에 그대로 삽입
```sql
SELECT
   name,
   price,
   (SELECT AVG(price) FROM products) AS avg_price
FROM
   products;
```
   - avg_price는 스칼라 서브쿼리 : SELECT 에 사용하는 서브쿼리는 스칼라 서브쿼리만 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/8cdb85fd-18bc-4405-a9a0-ebf120f3605e">
</div>

   - 쿼리 실행 흐름
      + 데이터베이스는 메인쿼리를 실행하기 전에, SELECT 절의 스칼라 서브쿼리를 단 한 번 먼저 실행 : (SELECT AVG(price) FROM products) 167166.6667
      + 데이터베이스는 이 계산된 값을 기억
      + 메인쿼리(SELECT name, price, ... FROM products)가 실행
      + products 테이블의 각 행을 가져올 때마다, avg_price 컬럼에 미리 계산해 둔 값(167166.6667)을 그대로 추가

   - 이처럼 서브쿼리가 외부 쿼리의 컬럼을 참조하지 않아 독립적으로 실행될 수 있는 경우를 앞서 배웠듯이 비상관 서브쿼리(Non-correlated Subquery) : 서브쿼리가 먼저 한 번 실행되고, 그 결과가 메인쿼리의 모든 행에 재사용되는 방식
   - 결과를 보면 모든 상품 행에 동일한 전체 평균 가격이 avg_price 컬럼으로 잘 추가된 것을 확인할 수 있음

3. 상관 서브쿼리
   - 비상관 서브쿼리는 유용하지만, SELECT 절의 스칼라 서브쿼리의 진정한 강력함은 메인쿼리의 각 행과 상호작용할 때 드러남
   - 전체 상품 목록을 조회하면서, 각 상품별로 총 몇 번의 주문이 있었는지 '총 주문 횟수'를 함께 보여주고 싶을 경우
   - '전체 평균'처럼 고정된 값이 아니라, '프리미엄 게이밍 마우스'의 주문 횟수, '기계식 키보드'의 주문 횟수 등 각 상품 행에 따라 계산 결과가 달라져야 하는 값이 필요
<div align="center">
<img src="https://github.com/user-attachments/assets/176170de-ef13-4a1b-9134-63e434ede3c6">
</div>

   - 이 문제를 해결하기 위해, 우리는 products 테이블의 각 상품 행을 하나씩 보면서, 그 상품이 orders 테이블에서 몇 번이나 주문되었는지를 개별적으로 세어야 함
   - 여기서 각 상품별 총 주문 횟수를 구하려면 다음과 같이 주문 번호가 서로 다른 서브쿼리를 실행
```sql
SELECT COUNT(*) FROM orders o WHERE o.product_id = 1;
SELECT COUNT(*) FROM orders o WHERE o.product_id = 2;
SELECT COUNT(*) FROM orders o WHERE o.product_id = 3;
SELECT COUNT(*) FROM orders o WHERE o.product_id = 4;
SELECT COUNT(*) FROM orders o WHERE o.product_id = 5;
SELECT COUNT(*) FROM orders o WHERE o.product_id = 6;
```
   - 정리하면 메인쿼리의 각 행마다 다음 서브쿼리를 추가로 실행해야 함
   - 그리고 {product_id} 값은 메인쿼리의 각 행마다 다른 product_id 값을 사용하면 됨
```sql
SELECT COUNT(*) FROM orders o WHERE o.product_id = {product_id}
```
   - 서브쿼리의 중요한 특징은, 바깥쪽 메인쿼리의 각 행마다 개별적으로, 반복적으로 실행될 수 있다는 점
   - 서브쿼리가 메인쿼리의 컬럼 값을 참조하는 관계를 가질 때, 이를 상관 서브쿼리(Correlated Subquery)
   - 메인쿼리는 products 테이블에서 상품 정보를 가져오는 것 :그리고 SELECT 절에 '총 주문 횟수'를 계산하는 서브쿼리를 하나의 컬럼처럼 추가
```sql
SELECT
   p.product_id,
   p.name,
   p.price,
   (SELECT COUNT(*) FROM orders o WHERE o.product_id = p.product_id) AS order_count
FROM
   products p;
```
   - 여기서 가장 중요한 부분은 서브쿼리 안의 WHERE o.product_id = p.product_id 조건.
   - p.product_id 는 메인쿼리(FROM products p)가 현재 처리하고 있는 행의 product_id 값을 의미
<div align="center">
<img src="https://github.com/user-attachments/assets/8a2ba462-8cad-4124-bcdb-677e00711860">
</div>

   - 쿼리 실행 흐름
      + 메인쿼리가 products 테이블의 첫 번째 행, '프리미엄 게이밍 마우스'(p.product_id =1)를 읽음
      + 이 행의 order_count 값을 계산하기 위해 스칼라 서브쿼리가 실행 : 이때 p.product_id 에는 1이 전달 (SELECT COUNT(```*```) FROM orders o WHERE o.product_id = 1)
      + 이 서브쿼리는 orders 테이블에서 product_id가 1인 주문을 세어 3이라는 단일 값을 반환
      + 첫 번째 행의 order_count 컬럼에는 3이 기록
      + 메인쿼리가 두 번째 행, '기계식 키보드'(p.product_id =2)를 읽음
      + 다시 스칼라 서브쿼리가 실행 : 이번에는 p.product_id 에 2가 전달 (SELECT COUNT(```*```) FROM orders o WHERE o.product_id = 2)
      + 서브쿼리는 1을 반환하고, 두 번째 행의 order_count 에는 1이 기록
      + 이 과정이 products 테이블의 모든 행에 대해 반복

   - 결과를 보면 각 상품별로 정확한 주문 횟수가 새로운 컬럼처럼 조회된 것을 볼 수 있음
   - 한 번도 팔리지 않은 상품의 주문 횟수는 0 으로 표시

4. 실무 팁 : 성능에 주의하라
   - 스칼라 서브쿼리는 이처럼 JOIN으로는 표현하기 복잡한 로직을 직관적으로 표현할 수 있게 해주지만, 강력한 만큼 주의해서 사용
   - 가장 큰 단점은 성능 저하의 가능
     + 특히 상관 서브쿼리는 메인쿼리가 반환하는 행의 수만큼 서브쿼리가 반복 실행되기 때문임
     + 만약 products 테이블에 100만 개의 상품이 있다면, 주문 횟수를 알기 위해 COUNT(```*```) 쿼리가 100만 번이나 실행되는 셈
     + 이는 데이터베이스에 엄청난 부하를 줄 수 있음
     + 사실 LEFT JOIN과 GROUP BY를 사용해서도 해결할 수 있으며, 대부분의 경우 데이터베이스 옵티마이저가 JOIN을 더 효율적으로 처리하여 성능이 더 좋음
```sql
-- JOIN으로 해결하는 방법
SELECT p.product_id, p.name, p.price, COUNT(o.order_id) AS order_count
FROM products p
LEFT JOIN orders o ON p.product_id = o.product_id
GROUP BY p.product_id, p.name, p.price;
```
   - 그럼에도 불구하고 스칼라 서브쿼리는 JOIN 이 너무 복잡해지거나, 완전히 다른 테이블에서 간단한 정보 하나만 조회해 올 때 코드를 훨씬 명료하게 만들어주는 장점이 있어 적재적소에 사용하면 매우 유용
