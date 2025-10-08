-----
### 상관 서브쿼리
-----
1. 한 번이라도 주문된 상품 조회하기
   - 쇼핑몰의 수많은 상품 중에 고객들이 어떤 상품을 주로 구매하는지 파악하는 것은 매우 중요
   - 가장 기본적인 분석은 "지금까지 한 번이라도 주문된 적이 있는 상품"이 무엇인지 알아보는 것
   - products 테이블에는 모든 상품 정보가 있고, orders 테이블에는 주문 기록이 있음 : 이 두 테이블을 함께 사용해서 이 문제를 해결
   - 문제 상황 : products 테이블에는 있지만, orders 테이블에는 한 번도 등장하지 않은 상품, 즉 '재고'로만 남아있는 상품을 제외하고, 실제 주문이 발생한 상품의 이름과 가격을 조회

2. 직접 확인하는 해결 방법
   - products 테이블
```sql
SELECT product_id, name, price
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/30da3247-15d9-4674-8f7f-74ccd4cb2aab">
</div>

   - 주문 내역이 있는 상품
```sql
SELECT DISTINCT product_id
FROM orders;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f6585136-cf3d-40ce-8e20-0334d32505ed">
</div>

   - 이제 product_id 기준으로 1, 2, 3, 4 제품만 찾으면 됨
   - products 테이블
```sql
SELECT product_id, name, price
FROM products
WHERE product_id IN (1, 2, 3, 4);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/23d6c2ec-4c62-47c4-afda-f4700b3e652f">
</div>

3. IN을 사용한 해결 방법
   - IN 연산자는 특정 컬럼의 값이 괄호 안에 있는 목록에 포함되는지를 확인하는 역할
   - "주문된 상품 ID 목록"을 먼저 구하고, products 테이블에서 해당 ID를 가진 상품들을 찾아낼 수 있음
   - 단계적 접근
      + 주문된 상품 ID 목록 조회 : 먼저 orders 테이블에서 어떤 상품들이 주문되었는지 product_id를 모두 가져옴 (이 때, 중복된 ID가 여러 개 있을 수 있으니 DISTINCT를 사용해 고유한 product_id 목록을 만듬)
      + 상품 정보 조회 : products 테이블에서 product_id가 1단계에서 만든 목록 안에 IN 하는 상품들만 조회
   - 쿼리 작성
```sql
SELECT
   product_id,
   name,
   price
FROM
   products
WHERE
   product_id IN (SELECT DISTINCT product_id
                  FROM orders);
```
   - 쿼리 실행 흐름
     + 데이터베이스는 먼저 서브쿼리인 SELECT DISTINCT product_id FROM orders를 실행
     + orders 테이블에 기록된 모든 주문을 확인하여, 주문된 상품들의 고유한 ID 목록을 만듬
       * orders 테이블에는 product_id가 1, 4, 2, 4, 3, 1, 1인 주문들이 존재
       * DISTINCT 를 통해 중복이 제거된 최종 목록은 (1, 2, 3, 4) 가 된다.
     + 이제 메인쿼리가 실행
       * SELECT product_id, name, price FROM products WHERE product_id IN (1, 2, 3, 4);
     + products 테이블의 모든 상품을 하나씩 확인하며 product_id가 (1, 2, 3, 4) 목록에 포함되는지 검사
       * product_id가 1인 '프리미엄 게이밍 마우스'는 목록에 있으므로 결과에 포함
       * product_id가 5인 '고급 가죽 지갑'은 목록에 없으므로 결과에서 제외
       * product_id가 6인 '스마트 워치'도 목록에 없으므로 결과에서 제외
<div align="center">
<img src="https://github.com/user-attachments/assets/5801145b-e157-46ae-ae04-695b229f376e">
</div>

   - IN을 사용하면 이처럼 원하는 결과를 직관적으로 얻을 수 있음
   - 이 방법은 대부분의 경우에 아주 효율적인 방법

4. EXISTS를 사용한 더 효율적인 방법
   - IN 방식은 orders 테이블이 매우 클 경우, 즉 주문량이 수백만, 수천만 건에 달하면 성능 문제를 일으킬 수 있음
   - orders를 조회하는 서브쿼리가 반환하는 지금까지 주문한 product_id 목록 전체를 메모리에 저장한 뒤, 메인쿼리의 각 행과 비교해야 하기 때문임
   - 이럴 때 EXISTS를 사용하면 효율적으로 쿼리를 실행할 수 있음
      + 💡 EXISTS는 서브쿼리가 반환하는 결과값 자체에는 관심이 없고, 오직 서브쿼리의 결과로 행이 하나라도 존재하는지 여부만 체크
   - EXISTS 연산
      + 서브쿼리 결과 행이 1개 이상이면 TRUE
      + 서브쿼리 결과 행이 0개이면 FALSE
   - 💡  EXISTS는 서브쿼리가 결과를 반환하는지 여부(Existence)만 확인
      + 서브쿼리가 단 하나의 행이라도 반환하면 TRUE, 아무 행도 반환하지 않으면 FALSE
```sql
SELECT
     product_id,
     name,
     price
FROM
     products p
WHERE EXISTS (SELECT 1
              FROM orders o
              WHERE o.product_id = p.product_id);
```
   - 여기서 p 와 o 는 각각 products 와 orders 테이블의 별칭
   - EXISTS 안의 서브쿼리를 보면 WHERE o.product_id = p.product_id 라는 조건이 존재 : 여기서는 상관 서브쿼리를 사용
   - 서브쿼리가 독립적으로 실행되지 않고 메인쿼리의 p 테이블 값에 의존하여 실행된다는 의미
   - 또한, 서브쿼리가 SELECT 1 을 사용하는 것을 볼 수 있음 : EXISTS는 결과 데이터가 무엇인지는 전혀 신경 쓰지 않고, 행이 존재하는지 여부만 보기 때문에 관례적으로 SELECT 1 과 같이 상수를 사용해 불필요한 데이터 조회를 피함
<div align="center">
<img src="https://github.com/user-attachments/assets/f071bea1-9538-4129-826c-281263d5e986">
</div>

   - 쿼리 실행 흐름 : EXISTS의 실행 흐름은 IN과 완전히 다름
     + 메인쿼리가 products (p) 테이블의 첫 번째 행인 '프리미엄 게이밍 마우스'(p.product_id = 1)를 읽음
     + 이 p.product_id 값을 가지고 서브쿼리가 실행 : SELECT 1 FROM orders o WHERE o.product_id = 1
     + orders 테이블에는 product_id 가 1인 주문이 3개 존재
        * 💡 데이터베이스는 조건을 만족하는 첫 번째 행을 찾자마자 더 이상 테이블을 탐색하지 않고, 서브쿼리가 결과를 반환할 수 있다고 판단 (예를 들어서 product_id 가 1인 주문이 10000개 존재하더라도 EXISTS는 첫 번째 행만 찾으면 바로 TRUE를 반환, 따라서 나머지 9999개를 찾지 않아도 됨)
     + EXISTS는 TRUE
     + WHERE TRUE 조건이 충족되었으므로, '프리미엄 게이밍 마우스'는 최종 결과에 포함
     + ... p.product_id = 2, 3, 4의 같은 내용이 반복 : 이 내용은 최종 결과에 포함
     + 메인쿼리가 products 테이블의 다음 행인 '고급 가죽 지갑'(p.product_id = 5)을 읽음
     + 다시 서브쿼리가 실행 : SELECT 1 FROM orders o WHERE o.product_id = 5
     + orders 테이블을 탐색했지만 product_id 가 5인 주문은 존재하지 않음 : 서브쿼리는 아무런 행도 반환하지 못함
     + EXISTS는 FALSE
     + WHERE FALSE 조건이므로, '고급 가죽 지갑'은 최종 결과에서 제외
     + 이후에 p.product_id = 6도 WHERE FALSE 조건이 되면서 최종 결과에서 제외
   - 결과적으로 이 과정을 products 테이블의 모든 행에 대해 반복

5. IN vs. EXISTS : 실무
<div align="center">
<img src="https://github.com/user-attachments/assets/08b7b7fd-a00c-46a5-b59e-805ee407782b">
</div>

  - 결론적으로, 서브쿼리의 대상이 되는 테이블(orders)이 크다면 EXISTS 가 유리
  - IN은 목록 전체를 비교해야 하지만, EXISTS는 조건을 만족하는 데이터를 '발견'하는 즉시 다음으로 넘어감

6. NOT EXISTS : 존재하지 않음을 확인하기
   - 반대로, 특정 조건의 데이터가 존재하지 않는 것을 확인하고 싶을 때는 NOT EXISTS를 사용
   - 문제 상황 : 한 번도 주문된 적이 없는, 이른바 '악성 재고' 상품을 찾기
```sql
SELECT
   product_id,
   name,
   price,
   stock_quantity
FROM
   products p
WHERE NOT EXISTS (SELECT 1
                  FROM orders o
                  WHERE o.product_id = p.product_id);
```
   - 이 쿼리는 위 EXISTS 예제와 정반대로 동작하여, 서브쿼리의 결과가 0건일 때 TRUE 를 반환
<div align="center">
<img src="https://github.com/user-attachments/assets/620a719c-db41-484d-b509-4b516df6f3d6">
</div>

7. 상관 서브쿼리와 성능
   - 상관 서브쿼리는 복잡한 로직을 매우 직관적으로 표현할 수 있게 해주지만, 성능에 주의
   - 메인쿼리의 행 수만큼 서브쿼리가 반복 실행될 수 있기 때문에, 메인쿼리가 다루는 데이터의 양이 많아지면 쿼리 전체의 성능이 급격히 저하될 수 있음
   - 많은 경우, 상관 서브쿼리는 JOIN(특히 LEFT JOIN 과 GROUP BY)으로 동일한 결과를 얻도록 재작성할 수 있으며, 데이터베이스 옵티마이저가 JOIN 을 더 효율적으로 처리하는 경우가 많음
   - 하지만 EXISTS는 특정 조건에 맞는 데이터가 있는지 '확인만 하고 넘어가는' 특성 덕분에, IN 이나 JOIN 보다 훨씬 효율적으로 동작하는 상황도 많음
   - 결론적으로 상관 서브쿼리는 성능 이슈를 인지하고, JOIN으로 표현하기 너무 복잡하거나 EXISTS를 통해 더 효율적인 실행이 가능할 때 적절히 사용하는 것이 중요
