-----
### 서브쿼리 소개
-----
1. 우리 쇼핑몰에서 판매하는 상품들의 평균 가격보다 비싼 상품은 무엇이 있을까?
   - 1단계 : 먼저, 전체 상품의 평균 가격을 구함 (AVG() 집계 함수를 사용하면 간단히 구할 수 있음)
```sql
SELECT AVG(price)
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/3c40b641-2c0f-42b3-bad7-9bedcf07f9ff">
</div>

   - 평균 가격이 약 167,167.67원이라는 것을 알 수 있음
   - 2단계 : 그 평균 가격보다 비싼 상품을 찾음 (이제 위에서 구한 값을 WHERE 절에 직접 넣어서 쿼리를 실행)
```sql
SELECT name, price
FROM products
WHERE price > 167166.67;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/9a119c22-b904-4b8d-898c-98ca734714cb">
</div>

2. 두 번에 걸쳐 쿼리를 실행하는 방식은 몇 가지 문제점을 가짐
   - 번거로움 : 매번 첫 번째 쿼리의 결과를 복사해서 두 번째 쿼리에 붙여 넣어야 함
   - 오류에 취약함 : 만약 상품 데이터가 실시간으로 추가되거나 가격이 변경될 경우
     + 1단계 쿼리를 실행하고 2단계 쿼리를 실행하는 그 짧은 순간에도 평균 가격은 변할 수 있음
     + 이렇게 되면 잘못된 기준으로 데이터를 조회하게 될 수도 있음
   - 이 두 단계를 논리적으로 완벽한 하나의 작업 단위로 묶고 싶은데, 이럴 때 사용하는 기술이 바로 서브쿼리(Subquery)

3. 서브쿼리의 개념
   - 서브쿼리는 말 그대로 하나의 SQL 쿼리 문 안에 포함된 또 다른 SELECT 쿼리를 의미
   - 바깥쪽의 메인쿼리가 실행 되기 전에, 괄호 () 안에 있는 서브쿼리가 먼저 실행
   - 그리고 데이터베이스는 서브쿼리의 실행 결과를 바깥쪽 메인 쿼리에게 전달하여, 메인쿼리가 그 결과를 사용해서 최종 작업을 수행
```sql
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```
   - 해당 쿼리의 실행 순서
     + 데이터베이스는 괄호 안의 서브쿼리, SELECT AVG(price) FROM products를 가장 먼저 실행
     + 서브쿼리가 실행된 결과인 167166.67 이라는 단일 값을 얻음
     + 이제 원래의 쿼리는 내부적으로 다음과 같이 변함 (SELECT name, price FROM products WHERE price > 167166.67;)
     + 최종적으로 이 변환된 메인쿼리가 실행되어 우리에게 결과를 보여줌
   - 결과는 우리가 두 단계로 나누어 실행했을 때와 완벽하게 동일하지만, 이제 단 하나의 쿼리로 해결

4. 서브쿼리 종류와 특징
   - 서브쿼리는 방금 본 WHERE절 외에도 쿼리의 다양한 위치에서 활약하며 각기 다른 역할을 수행
   - 서브쿼리는 반환하는 행과 컬럼의 수에 따라 종류가 나뉘며, 사용되는 위치와 연산자에 따라 그 역할이 결정
<div align="center">
<img src="https://github.com/user-attachments/assets/6ab7fdc6-7a3e-4902-b7cf-d89117c82c76">
</div>

   - 테이블 서브쿼리는 표에는 다중 컬럼 다중 행으로 표기했지만 컬럼, 행 수가 단일이어도 쓸 수 있음
   - 실무에선 통상 다중 컬럼, 다중 행에 자주 사용
