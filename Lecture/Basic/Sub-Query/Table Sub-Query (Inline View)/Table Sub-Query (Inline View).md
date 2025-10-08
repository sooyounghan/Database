-----
### 테이블 서브쿼리
-----
1. FROM 절에 위치하는 서브쿼리는, 그 실행 결과가 마치 하나의 독립된 가상 테이블처럼 사용되기 때문에 테이블 서브쿼리라고 함
   - 쿼리 내에서 '인라인(inline)'으로 즉석에서 정의되는 '뷰(View, 가상 테이블)'와 같다고 해서 인라인 뷰(Inline View)라고도 불림

2. 테이블 서브쿼리의 가장 큰 특징은, 우리가 지금까지 배운 복잡한 SELECT 문(집계, 그룹핑, 조인 등)의 결과를 하나의 명확한 데이터 집합으로 먼저 만들어 놓고, 그 집합을 대상으로 다시 한번 SELECT를 할 수 있게 해준다는 점
3. 각 상품 카테고리별로, 가장 비싼 상품의 이름과 가격을 조회하고 싶은 경우
```sql
-- name이 빠짐
SELECT category, MAX(price)
FROM products
GROUP BY category;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e9e71a2b-4caf-4589-80f5-cef5855f3ff0">
</div>

   - 이름(name)을 추가
```sql
-- 잘못된 쿼리의 예
SELECT category, name, MAX(price)
FROM products
GROUP BY category;
```
   - 실행 결과
```
Error Code: 1055. Expression #2 of SELECT list is not in GROUP BY clause and
contains nonaggregated column 'my_shop2.products.name' which is not
functionally dependent on columns in GROUP BY clause; this is incompatible
with sql_mode=only_full_group_by
```
   - products.name 컬럼은 GROUP BY 절에 없기 때문에 사용할 수 없다는 오류
   - 위 쿼리를 실행하면, 데이터베이스는 GROUP BY로 묶인 category별로 MAX(price)는 정확하게 계산할 수 있지만, name은 해당 그룹의 여러 상품명 중에 어떤 것을 보여줄지 선택할 수 없으므로, 따라서 오류가 발생
   - 예를 들어서 전자기기 카테고리만 해도 각각 다른 4개의 상품명이 존재하는데, 이 중에 어떤 상품명 하나를 선택해야 할지 적절한 기준이 없으므로 오류가 발생
```sql
SELECT name, category, price FROM products
WHERE category = '전자기기';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/96ef70a8-9b25-4135-a25e-d9ff98586fac">
</div>

   - 원하는 것은 '각 카테고리별 최고가'를 먼저 알아낸 뒤, 그 가격과 일치하는 '바로 그 상품'을 찾는 것
   - 이 2단계 로직을 구현하는 데 인라인 뷰가 적절한 해법을 제시

4. 인라인 뷰를 이용한 2단계 접근법
   - 1단계 (인라인 뷰) : 카테고리별 최고가격을 미리 구함
     + 먼저, 카테고리별로 가장 높은 가격이 얼마인지 알아내는 쿼리를 작성
     + 이 쿼리가 바로 인라인 뷰, 즉 가상 테이블이 될 것이다.
```sql
SELECT category, MAX(price) AS max_price
FROM products
GROUP BY category
```
<div align="center">
<img src="https://github.com/user-attachments/assets/6479fb5c-5968-4634-b2ee-a258189a556c">
</div>

   - 이 쿼리를 실행하면 '전자기기', '도서', '패션' 각 카테고리의 최고가격을 담은 3줄짜리 결과가 나옴
   - 이제 이 결과를 category_max_price 라는 이름의 가상 테이블이라고 생각

   - 2단계 (메인쿼리) : 원본 테이블과 가상 테이블을 조인
     + 원본 products 테이블과, 방금 만든 가상 테이블(category_max_price)을 조인
     + 연결 조건은 두 가지
        * 카테고리 이름이 같아야 함 (p.category = cmp.category)
        * 상품 가격이 그 카테고리의 최고가와 같아야 함 (p.price = cmp.max_price)
     + 이 두 조건을 모두 만족하는 상품만이 '카테고리별 최고가 상품'이라는 것을 보장할 수 있음

5. 최종 쿼리 작성
   - 이제 두 단계를 합쳐 최종 쿼리를 완성
   - FROM 절에 들어가는 서브쿼리에는 반드시 별칭(Alias)을 붙여줘야 한다는 점을 잊지 말 것
   - 여기서는 cmp(category_max_price)라는 별칭을 사용
```sql
SELECT
   p.product_id,
   p.name,
   p.price
FROM
   products p
JOIN
   (SELECT category, MAX(price) AS max_price
   FROM products
   GROUP BY category) AS cmp
ON
   p.category = cmp.category AND p.price = cmp.max_price;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a9689a73-9eb3-45dc-898b-ca775ed9688d">
</div>

6. 실행 결과 분석
   - 기준 테이블 : products
```sql
SELECT product_id, name, category, price
FROM products p;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/9da623b4-7a3a-4bc2-98e7-ddb916e082b6">
</div>

   - 조인 대상 테이블 - 인라인 뷰
```sql
SELECT category, MAX(price) AS max_price
FROM products
GROUP BY category;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/26b2b6cc-6349-401a-9434-37c08374faef">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/076809bf-4ee1-497d-9302-fc2f90f3ae2e">
</div>

   - 쿼리 실행 흐름
     + 데이터베이스는 FROM 절의 서브쿼리(인라인 뷰)를 먼저 실행하여, cmp 라는 임시 테이블을 메모리에 생성 : 이 테이블에는 각 카테고리와 그 카테고리의 최고 가격이 들어있음
     + 그다음, 메인쿼리가 실행 : products 테이블(별칭 p)과 방금 생성된 임시 테이블 cmp를 INNER JOIN
     + ON 절에 명시된 두 가지 조건(카테고리 일치, 가격 일치)을 모두 만족하는 행만 최종 결과로 선택

   - 결과를 보면 GROUP BY의 함정을 피하고, 각 카테고리별로 가장 비싼 상품의 정보를 정확하게 찾아냄

7. 이처럼 인라인 뷰는 복잡한 데이터를 단계적으로 가공해야 할 때, 특히 집계된 결과를 가지고 다시 한번 조인이나 필터링을 수행해야 할 때 매우 유용

8. FROM 절의 상관 서브쿼리 - LATERAL
   - FROM 절에서 상관 서브쿼리를 사용하려면 LATERAL 이라는 특별한 키워드를 사용해야 함
   - 이 기능은 너무 복잡하고, 성능도 잘 나오지 않는 문제가 있어서 실무에서는 잘 사용하지 않는 편
   - FROM 절에 상관 서브쿼리를 꼭 사용해야 하는 특별한 일이 있다면 LATERAL 키워드를 검색
