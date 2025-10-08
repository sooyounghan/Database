-----
### 다중 행 서브쿼리
-----
1. '전자기기' 카테고리에 속한 모든 상품들을 주문한 주문 내역을 전부 보고 싶을 경우
   - 먼저, '전자기기' 카테고리에 속한 상품들의 product_id를 모두 찾음
   - 그다음, orders 테이블에서 product_id가 우리가 찾아낸 product_id 목록 안에 포함된 주문들을 모두 찾음

2. 여기서 위 결과는 당연히 여러 개일 것 : 우리 쇼핑몰에는 여러 종류의 '전자기기' 상품이 있기 때문임
   - 이처럼 서브쿼리의 결과가 여러 행으로 반환되는 것이 당연할 때 사용하는 것이 바로 다중 행 서브쿼리
   - 다중 행 서브쿼리의 결과를 처리하기 위해서는 = 같은 단일 행 연산자가 아닌, 목록을 다룰 수 있는 특별한 연산자가 필요 (대표적으로 IN, ANY, ALL 등 존재)
   
3. IN 연산자 : 목록에 포함된 값과 일치하는지 확인
   - IN 연산자는 다중 행 서브쿼리와 함께 가장 흔하게 사용되는, 가장 직관적인 연산자
   - WHERE 컬럼명 IN (값1, 값2, ...) 처럼, 특정 컬럼의 값이 괄호 안의 목록 중 하나라도 일치하면 참(true)을 반환
   - 1단계 (서브쿼리) : '전자기기' 상품의 ID 목록 조회
```sql
SELECT product_id
FROM products
WHERE category = '전자기기'
ORDER BY product_id;
```
   - 확인의 편의상 정렬을 추가 (서브쿼리에 사용할 때는 정렬을 빼는 것이 좋음)
<div align="center">
<img src="https://github.com/user-attachments/assets/6febba10-2422-422a-94f3-c8ef8ef15163">
</div>

   - 예상대로 '전자기기' 카테고리 상품들의 id 목록 (1, 2, 3, 6) 이 반환
   - 2단계 (메인쿼리) : IN 을 이용해 최종 결과 조회
      + 이제 이 쿼리를 서브쿼리로 사용하여, orders 테이블에서 product_id가 위 목록에 포함된 주문들만 필터링
```sql
SELECT *
FROM orders
WHERE product_id IN (SELECT product_id
                     FROM products
                     WHERE category = '전자기기')
ORDER BY order_id;
```
   - 쿼리 실행 흐름
     + 괄호 안의 서브쿼리가 먼저 실행되어 (1, 2, 3, 6)이라는 id 목록을 반환
     + 메인쿼리는 WHERE product_id IN (1, 2, 3, 6)과 동일한 형태로 바뀜
     + 즉, product_id 가 1이거나, 2거나, 3이거나, 6인 모든 주문을 찾아냄
     + 최종적으로 메인쿼리가 실행
   - 쉽게 이야기하면 서브쿼리가 (1, 2, 3, 6)의 값 목록으로 변환되어서 다음 쿼리가 실행되는 것
```sql
SELECT * FROM orders
WHERE product_id IN (1,2,3,6)
ORDER BY order_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f3657669-8643-44a6-bbae-0cf1d9c6e7d1">
</div>

   - 이렇게 IN 연산자를 활용하여 여러 결과를 반환하는 서브쿼리를 깔끔하게 처리
   - 참고로 목록에 없는 것을 찾고 싶을 때는 NOT IN을 사용

4. ANY, ALL 연산자 : 목록의 모든 / 일부 값과 비교
   - ANY와 ALL은 주로 ```>, <``` 와 같은 비교 연산자와 함께 사용되어, 서브쿼리가 반환한 여러 값들과 비교하는 역할
      + ```>``` ANY (서브쿼리) : 서브쿼리가 반환한 여러 결과값 중 어느 하나보다만 크면 참 (즉, 최소값보다 크면 참)
      + ```>``` ALL (서브쿼리) : 서브쿼리가 반환한 여러 결과값 모두보다 커야만 참 (즉, 최대값보다 커야 참)
      + ```<``` ANY (서브쿼리) : 최대값보다 작으면 참
      + ```<``` ALL (서브쿼리) : 최소값보다 작으면 참
      + ```=``` ANY (서브쿼리) : IN 과 완전히 동일한 의미 (목록 중 어느 하나와 같으면 참)

   - 어떤 서브쿼리가 (100, 200, 300) 이라는 세 개의 값을 반환했다고 가정    
      + ```WHERE price > ANY (100, 200, 300)``` : 이 조건은 ```price > 100```과 같음 (100보다 크기만 하면 200이나 300보다 작더라도, 목록 중 100보다는 크기 때문에 참)
      + ```WHERE price > ALL (100, 200, 300)``` : 이 조건은 ```price > 300```과 같음 (목록의 모든 값, 즉 100, 200, 300보다 모두 커야 하므로, 결국 최대값인 300보다 커야 참)

5. 실제 쇼핑몰 데이터로 ANY , ALL 사용해보기
   - 문제 상황 1 : '전자기기' 카테고리의 어떤 상품보다도 비싼 상품 찾기
     + 이것이 바로 ```> ANY``` 를 사용하기 좋은 상황
     + '전자기기' 카테고리 상품들의 가격 목록 중, 어느 하나보다만 크면 되기 때문임
     + 먼저, 서브쿼리를 통해 '전자기기' 카테고리의 상품 가격들을 확인
```sql
SELECT price
FROM products
WHERE category = '전자기기';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/9f9ad87b-81b6-47c7-a40e-d7e64445028a">
</div>

   - 서브쿼리는 (75000, 120000, 350000, 280000)이라는 가격 목록을 반환
   - 이제 ANY 를 사용하여 이 목록의 최소값(75000)보다 비싼 모든 상품을 조회
```sql
SELECT name, price
FROM products
WHERE price > ANY (SELECT price
                    FROM products
                    WHERE category = '전자기기');
```
   - 이 쿼리는 ```price > 75000 OR price > 120000 OR price > 350000 OR price > 280000```와 같이 동작
   - 결국 price가 목록의 최소값인 75000보다 크기만 하면 이 조건이 참이 되므로, ```price > 75000```과 동일한 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/256d8f2c-dd73-48d4-9280-f49f998ecc75">
</div>

   - 결과를 보면 '프리미엄 게이밍 마우스'(75,000원)를 제외하고, 그보다 비싼 모든 상품이 조회된 것을 확인할 수 있음
   - 심지어 같은 '전자기기' 카테고리에 속한 '기계식 키보드', '4K UHD 모니터', '스마트 워치'도 포함

   - 문제 상황 2 : '전자기기' 카테고리의 모든 상품보다 비싼 상품 찾기
      + ```> ALL``` 을 사용할 완벽한 시나리오 : '전자기기' 카테고리 상품들의 모든 가격보다 커야 하기 때문임
```sql
SELECT name, price
FROM products
WHERE price > ALL (SELECT price
                    FROM products
                    WHERE category = '전자기기');
```
   - 서브쿼리는 이전과 동일하게 (75000, 120000, 350000, 280000)를 반환
   - ```> ALL 조건```은 이 모든 값보다 커야 하므로, 목록의 최대값인 350000보다 커야 참이 됨
   - 즉, ```price > 350000```과 동일하게 동작
   - 실행 결과 : 결과 없음

   - 현재 products 테이블에는 350,000원보다 비싼 상품이 없으므로 아무런 결과도 반환되지 않음

6. 실무 팁 : MIN , MAX 집계 함수와의 비교
   - ANY 와 ALL은 특정 조건(예) 특정 부서 직원들의 모든 급여보다 많은 급여를 받는 사람 찾기)을 해결하는 데 유용
   - 사실 실무에서는 IN 연산자나 MIN() , MAX() 같은 집계 함수를 이용한 서브쿼리로 대체할 수 있는 경우가 많음
   - 오히려 집계 함수를 쓰는 것이 코드가 더 명확하고 직관적일 때가 많음
   - 예를 들어, 앞서 작성한 ANY, ALL 쿼리는 다음과 같이 바꿀 수 있음
   - ```> ANY``` 쿼리 대체 (MIN 사용)
```sql
SELECT name, price
FROM products
WHERE price > (SELECT MIN(price)
              FROM products
              WHERE category = '전자기기');
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c9d403e8-87e5-48f9-ab66-f29b110f8353">
</div>

   - ```> ALL``` 쿼리 대체 (MAX 사용)
```sql
SELECT name, price
FROM products
WHERE price > (SELECT MAX(price)
              FROM products
              WHERE category = '전자기기');
```
   - 실행 결과 : (결과 없음)
   - 앞서 ALL 을 사용한 실행 결과와 같음

   - 두 쿼리 모두 ANY, ALL 을 사용했을 때와 완전히 동일한 결과를 반환
     + 많은 개발자들이 ANY, ALL 보다는 MIN, MAX 를 사용한 코드를 더 이해하기 쉽다고 생각
     + 그렇기 때문에 ANY, ALL 의 사용 빈도는 IN 이나 집계 함수에 비해 낮은 편

7. 우리가 지금까지 배운 서브쿼리는 결과로 하나의 값(단일 행, 단일 컬럼)을 반환하거나, 값의 목록(다중 행, 단일 컬럼)을 반환하는 경우
   - 그런데 만약 서브쿼리가 하나의 행을 반환하지만, 그 안에 여러 개의 컬럼을 담고 있다면 어떻게 해야 할까?
   - 바로 이럴 때 다중 컬럼 서브쿼리(Multi-Column Subquery)를 사용
   - 이름 그대로, 서브쿼리가 반환하는 결과에 여러 개의 컬럼이 포함되는 경우
