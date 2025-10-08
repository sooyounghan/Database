-----
### CASE문 기본
-----
1. CASE 문은 데이터 '자체'를 동적으로 가공하고 새로운 의미를 부여하는, 데이터에 멋진 옷을 입히는 기술
   - 마치 IF-THEN-ELSE 처럼, 특정 조건에 따라 다른 값을 출력하게 만드는 SQL의 강력한 조건부 로직 도구

2. 상황 : 상품 목록을 조회하는데, 그냥 가격만 보여주지 말고, 가격대에 따라 '고가', '중가', '저가'와 같은 알아보기 쉬운 등급을 옆에 함께 표시하고 싶음
   - 10만원 이상 : '고가'
   - 3만원 이상 10만원 미만 : '중가'
   - 3만원 미만 : '저가'

3. 물론 이 작업은 데이터베이스에서 원본 데이터를 모두 가져온 뒤, 자바나 파이썬 같은 애플리케이션 코드에서 if ~ else문으로 처리할 수도 있음
   - 하지만 CASE 문을 사용하면 간단한 보고서나 데이터 분석을 할 때, 쿼리 하나만으로 원하는 최종 결과물을 바로 얻을 수 있어 매우 편리함

4. CASE 문의 두 가지 종류
   - CASE 문은 크게 두 가지 방식으로 나눌 수 있음
     + 단순 CASE 문(Simple CASE Expression)
     + 검색 CASE 문(Searched CASE Expression)
     
5. 단순 CASE 문 (Simple CASE Expression)
   - 단순 CASE 문은 특정 하나의 컬럼이나 표현식의 값에 따라 결과를 다르게 하고 싶을 때 사용
   - 특정 값을 기준으로 '이 값이 A면 X, B면 Y, 그 외에는 Z'와 같이 명확하게 분기할 때 유용
   - 단순 CASE 문의 기본 문법
```sql
CASE 비교대상_컬럼_또는_표현식
   WHEN 값1 THEN 결과1
   WHEN 값2 THEN 결과2
   ...
   ELSE 그_외의_경우_결과
END
```
   - 비교대상_컬럼_또는_표현식 : WHEN 절에서 비교할 대상이 되는 컬럼 또는 표현식
   - WHEN 값 THEN 결과 : 비교대상_컬럼_또는_표현식의 값이 값과 같을 경우 결과를 반환
   - ELSE 결과 : 위에 명시된 WHEN 조건들 중 어느 것 하나도 참이 아닐 경우, ELSE 뒤의 결과를 반환
     + 💡 ELSE를 생략했는데 모든 WHEN 조건이 거짓이면 NULL이 반환
   - 실행 순서 : 단순 CASE 문도 위에서 아래 순서대로 조건을 평가하며, 가장 먼저 일치하는 WHEN 절을 만나는 순간 그 THEN의 결과를 반환하고 즉시 평가를 종료

6. 단순 CASE 문 예제 : 주문 상태(status)를 한글로 표시하기
   - 쇼핑몰의 orders 테이블에는 주문 상태를 PENDING, COMPLETED, SHIPPED, CANCELLED와 같은 영어로 저장하고 있음
   - 고객에게는 이 상태를 한글로 보여주고 싶음
```sql
SELECT
   order_id,
   user_id,
   product_id,
   quantity,
   status,
   CASE status
       WHEN 'PENDING' THEN '주문 대기'
       WHEN 'COMPLETED' THEN '결제 완료'
       WHEN 'SHIPPED' THEN '배송'
       WHEN 'CANCELLED' THEN '주문 취소'
       ELSE '알 수 없음' -- 예상치 못한 상태 값 처리
   END AS status_korean
FROM
   orders;
```
   - 이 쿼리는 orders 테이블의 각 행마다 status 컬럼의 값을 확인하여 해당하는 한글 상태 값을 status_korean이라는 새로운 컬럼으로 반환
<div align="center">
<img src="https://github.com/user-attachments/assets/850312bb-a4a7-40ae-a4ab-8cd4e081c34d">
</div>

   - 단순 CASE 문은 이렇게 하나의 기준 값에 따라 명확하게 분류할 때 깔끔하고 가독성이 좋음

7. 검색 CASE 문 (Searched CASE Expression)
   - 검색 CASE 문은 단순 CASE 문처럼 하나의 특정 값을 비교하는 대신, 각 WHEN 절에 독립적인 조건식을 사용하여 복잡한 논리를 구현할 때 사용
   - 앞서 만난 문제 상황처럼 '가격이 얼마 이상', '날짜가 언제 이전'과 같은 범위 조건이나 여러 컬럼을 조합한 복합적인 조건이 필요할 때 주로 사용
   - 검색 CASE 문의 기본 문법
```sql
CASE
   WHEN 조건1 THEN 결과1
   WHEN 조건2 THEN 결과2
   ...
   ELSE 그_외의_경우_결과
END
```
   - WHEN 조건 THEN 결과 : WHEN 뒤의 조건이 참(true)일 경우, THEN 뒤의 결과를 반환
      + 여기서 조건은 price >= 100000 , category = '도서' 등 다양한 비교 연산자와 논리 연산자(AND, OR, NOT)를 포함할 수 있음
   - ELSE 결과 : 위에 명시된 WHEN 조건들 중 어느 것 하나도 참이 아닐 경우, ELSE 뒤의 결과를 반환
      + 💡 ELSE를 생략했는데 모든 WHEN 조건이 거짓이면 NULL이 반환
   - 실행 순서 : 검색 CASE 문도 위에서 아래 순서대로 조건을 평가하며, 가장 먼저 참이 되는 WHEN 절을 만나는 순간 그 THEN의 결과를 반환하고 즉시 평가를 종료 (이 순서가 매우 중요)
   - 검색 CASE 문 예제 : 상품 가격에 따라 등급 표시하기
      + 상품 가격에 따라 '고가', '중가', '저가' 등급을 표시하는 문제를 검색 CASE 문으로 해결해보기
```sql
SELECT
   name,
   price,
   CASE
     WHEN price >= 100000 THEN '고가'
     WHEN price >= 30000 THEN '중가'
     ELSE '저가'
   END AS price_label
FROM
 products;
```
   - 여기서 AS price_label을 사용해, CASE 문 전체가 만들어내는 결과에 price_label이라는 새로운 컬럼명을 붙여줌
   - 이 쿼리는 products 테이블의 각 행마다 다음의 로직을 수행
     + 가격이 10만원 이상인가? 맞으면 '고가'를 반환하고 CASE 문을 끝냄
     + (1번이 아니라면) 가격이 3만원 이상인가? 맞으면 '중가'를 반환하고 끝냄
     + (1, 2번이 모두 아니라면) ELSE에 따라 '저가'를 반환

<div align="center">
<img src="https://github.com/user-attachments/assets/57ed5514-10e5-4c42-ba7c-45b472e1c77e">
</div>

   - 보는 것과 같이, 원본 데이터는 전혀 건드리지 않고, 조회 결과에만 우리가 정의한 비즈니스 로직에 따라 새로운 값을 동적으로 생성하여 보여주었음

-----
### 주의사항
-----
1. 검색 CASE 문 사용 시 주의사항 : WHEN 절의 순서
   - CASE 문은 위에서 아래로 순차적으로 평가하며, 가장 먼저 참이 되는 조건을 만나는 순간 실행을 멈춘다고 강조
     + 이 점이 검색 CASE 문에서는 특히 중요
     + 만약 조건을 잘못 배치하면 예상과 다른 결과가 나올 수 있음
   - 만약 위 쿼리에서 WHEN price >= 30000 조건을 WHEN price >= 100000 조건보다 먼저 배치할 경우
```sql
-- 잘못된 순서의 예 (의도와 다른 결과)
SELECT
   name,
   price,
   CASE
       WHEN price >= 30000 THEN '중가' -- 이 조건이 먼저 평가
       WHEN price >= 100000 THEN '고가'
       ELSE '저가'
   END AS price_label
FROM
   products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/fbdd48a6-f9f4-43cf-8cbf-1a631c2f84d7">
</div>

   - 프리미엄 게이밍 마우스 (75000원) : 75000원은 30000원 이상이므로 '중가'로 분류 (올바름)
   - 기계식 키보드 (120000원) : 120000원은 30000원 이상이므로 '중가'로 분류 (틀림, '고가'여야 함)
   - 이처럼 WHEN price >= 100000인 상품도 price >= 30000 조건을 먼저 만족시켜 '중가'로 잘못 분류될 것
   - 따라서 검색 CASE 문을 사용할 때는 조건의 순서를 신중하게 고려해야 함 : 더 포괄적인(범위가 넓은) 조건보다는 더 구체적인(범위가 좁은) 조건을 먼저 배치하는 것이 일반적

2. CASE 문과 사용 위치
   - CASE 문은 SELECT 절 외에도 ORDER BY, GROUP BY, WHERE 절 등 다양한 SQL 구문과 함께 사용될 수 있음
   - 예를 들어, 상품을 '고가', '중가', '저가' 순서로 정렬하고 싶다면 ORDER BY 절에 CASE 문을 사용할 수 있음
```sql
SELECT
     name,
     price,
     CASE
         WHEN price >= 100000 THEN '고가'
         WHEN price >= 30000 THEN '중가'
         ELSE '저가'
     END AS price_label
FROM
     products
ORDER BY
     CASE
         WHEN price >= 100000 THEN 1 -- 고가 : 1
         WHEN price >= 30000 THEN 2 -- 중가 : 2
         ELSE 3 -- 저가 : 3
     END ASC, -- 숫자가 작은 순서대로 정렬
     price DESC; -- 같은 등급 내에서는 가격 내림차순
```
<div align="center">
<img src="https://github.com/user-attachments/assets/7e6dd662-a673-4983-a2e8-d67ceed5ecda">
</div>

   - 보는 것처럼, price_label 컬럼의 '고가', '중가', '저가' 문자열 순서가 아닌, CASE 문에서 지정한 1, 2, 3의 숫자 순서대로 정렬된 것을 확인할 수 있음
   - 이렇게 CASE 문은 데이터의 표현뿐만 아니라 정렬, 그룹화 등 다양한 로직에 활용할 수 있음
