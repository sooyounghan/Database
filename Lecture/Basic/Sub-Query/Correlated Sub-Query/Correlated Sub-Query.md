-----
### 상관 서브쿼리
-----
1. 각 상품별로, 자신이 속한 카테고리의 평균 가격 이상의 상품들을 찾기
   - 자신이 속한 바로 그 카테고리의 평균 가격과 비교해야 한다는 점이 핵심
   - '프리미엄 게이밍 마우스'는 '전자기기' 카테고리에 속함 : '전자기기' 카테고리의 평균 가격과 비교해야 함
   - '관계형 데이터베이스 입문' 책은 '도서' 카테고리에 속함 : '도서' 카테고리의 평균 가격과 비교해야 함
   - '고급 가죽 지갑'은 '패션' 카테고리에 속함 : '패션' 카테고리의 평균 가격과 비교해야 함

2. 이처럼 서브쿼리가 메인쿼리에서 현재 처리 중인 행의 특정 값(예) 카테고리명)을 알아야만 계산을 수행할 수 있을 때, 바로 상관 서브쿼리(Correlated Subquery)를 사용
   - 여기서 상관(Correlated)의 의미는 메인쿼리와 서브쿼리가 서로 영향을 준다는 뜻
   - 이 문제를 해결하려면, products 테이블의 각 행을 하나씩 확인하면서 비교 대상을 동적으로 바꿔야 함
<div align="center">
<img src="https://github.com/user-attachments/assets/bd51f56f-c18e-4688-9ba7-327fec805bb4">
</div>

   - 위 표에서 볼 수 있듯이, 우리는 products 테이블의 모든 상품을 하나씩 훑어봐야 함
   - 그리고 각 상품에 대해, 그 상품이 속한 카테고리의 평균 가격을 그때그때 계산해서 현재 상품의 가격과 비교해야 함
   - 따라서 다음과 같이 각각의 상품마다 다른 서브쿼리를 실행해야 함
```sql
SELECT AVG(price) FROM products WHERE category = '전자기기' -- product_id:1
SELECT AVG(price) FROM products WHERE category = '전자기기' -- product_id:2
SELECT AVG(price) FROM products WHERE category = '전자기기' -- product_id:3
SELECT AVG(price) FROM products WHERE category = '도서' -- product_id:4
SELECT AVG(price) FROM products WHERE category = '패션' -- product_id:5
SELECT AVG(price) FROM products WHERE category = '전자기기' -- product_id:6
```
   - 정리하면 메인쿼리의 각 행마다 다음 서브쿼리를 추가로 실행해야 함
   - 그리고 {category} 값은 메인쿼리의 각 행마다 다른 category 값을 사용해야 함
```sql
SELECT AVG(price)
FROM products
WHERE category = {category}
```
   - 기존의 비상관 서브쿼리처럼 서브쿼리가 한 번만 실행되어서는 이 문제를 해결할 수 없음
   - 메인쿼리가 어떤 상품을 보고있는지에 따라 서브쿼리의 계산 결과가 계속 달라져야 하기 때문임

3. 상관 서브쿼리의 개념
   - 상관 서브쿼리는 이름 그대로, 메인쿼리와 서브쿼리가 서로 '연관' 관계를 맺고 동작하는 서브쿼리
   - 서브쿼리가 독립적으로 실행될 수 없고, 메인쿼리의 값을 참조하여 실행되는 것이 특징
   - 상관 서브쿼리의 동작 방식은 기존 서브쿼리와 완전히 다름
   - 비상관 서브쿼리 (Non-correlated) : 서브쿼리가 단 한 번 실행된 후, 그 결과를 메인쿼리가 사용
   - 상관 서브쿼리 (Correlated)
     + 메인쿼리가 먼저 한 행을 읽음
     + 읽혀진 행의 값을 서브쿼리에 전달하여, 서브쿼리가 실행
     + 서브쿼리 결과를 이용해 메인쿼리의 WHERE 조건을 판단
     + 메인쿼리의 다음 행을 읽고, 2-3번 과정을 반복
   - 즉, 서브쿼리가 메인쿼리의 행 수만큼 반복 실행될 수 있음
   
4. 상관 서브쿼리로 문제 해결하기
   - 각 상품별로, 자신이 속한 카테고리의 평균 가격 이상의 상품들을 찾기
   - 먼저, 각 카테고리별 평균 가격을 직접 계산
      + 전자기기 : ( 75000 + 120000 + 350000 + 280000 ) / 4 = 206,250원
      + 도서 : 28000 원 (상품이 1개)
      + 패션 : 150000 원 (상품이 1개)
   - 이 값들을 기준으로 각 상품의 가격을 비교해야 함
```sql
SELECT
   product_id,
   name,
   category,
   price
FROM
   products p1
WHERE
   price >= (SELECT AVG(price)
             FROM products p2
             WHERE p2.category = p1.category);
```
   - 이 쿼리에서 가장 중요한 부분은 서브쿼리 안의 WHERE p2.category = p1.category 
      + p1은 메인쿼리의 products 테이블을 가리키는 별칭
      + p2는 서브쿼리의 products 테이블을 가리키는 별칭
      + 이 조건문은 "서브쿼리에서 평균을 계산할 때, 메인쿼리가 현재 보고 있는 상품(p1)과 동일한 카테고리를 가진 상품들(p2)만을 대상으로 하라"는 의미 (이것이 바로 '연관'의 핵심)

   - 쿼리 실행 흐름
<div align="center">
<img src="https://github.com/user-attachments/assets/1adf323a-be35-448f-a9a4-fc8879195600">
</div>

   - 메인쿼리는 우선 products 행들을 읽음
   - 그리고 각 행에 대해서 순서대로 다음 로직을 수행
      + 메인쿼리가 products (p1)의 첫 행 '프리미엄 게이밍 마우스'(가격 75000, 카테고리 '전자기기')를 읽음 (이 때, p1.category는 '전자기기')
      + 이 값을 받아 서브쿼리가 실행 : SELECT AVG(price) FROM products p2 WHERE p2.category='전자기기'이므로, 결과는 206250
      + 메인쿼리의 WHERE 절은 WHERE 75000 >= 206250 으로 바뀜 (이 조건은 거짓(false)이므로 이 행은 결과에 포함되지 않음)
      + 메인쿼리가 다음 행 '기계식 키보드'(가격 120000, 카테고리 '전자기기')를 읽음
      + 서브쿼리가 다시 실행 : p1.category는 여전히 '전자기기'이므로, SELECT AVG(price) FROM products p2 WHERE p2.category = '전자기기'가 실행되고 결과는 206250
      + WHERE 절은 WHERE 120000 >= 206250으로 바뀌지만, 거짓이므로 이 행도 포함되지 않음
      + 메인쿼리가 다음 행 '4K UHD 모니터'(가격 350000, 카테고리 '전자기기')를 읽음
      + 서브쿼리는 또 '전자기기'의 평균 가격인 206250을 반환
      + WHERE 절은 WHERE 350000 >= 206250으로 바뀜 : 참(true)이므로 이 행은 최종 결과에 포함
      + 메인쿼리가 다음 행 '관계형 데이터베이스 입문'(가격 28000, 카테고리 '도서')을 읽음 : p1.category는 이제 '도서'
      + 서브쿼리가 실행 : SELECT AVG(price) FROM products p2 WHERE p2.category = '도서'이며, 결과는 28000
      + WHERE 절은 WHERE 28000 >= 28000으로 바뀌고, 참이므로 이 행도 결과에 포함
      + 이 과정이 products 테이블의 모든 행에 대해 반복
<div align="center">
<img src="https://github.com/user-attachments/assets/5ea6c8ee-92b0-4c30-83d3-6d262623dc6d">
</div>

   - 결과를 보면, 각자 자신이 속한 카테고리의 평균 가격(전자기기 : 206,250원, 도서 : 28,000원, 패션 : 150,000원)과 이상인 상품들만 정확하게 조회된 것을 확인 가능
