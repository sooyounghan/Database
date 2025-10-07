-----
### 집계 함수
-----
1. SQL은 강력하고 편리한 집계 함수(Aggregate Functions)를 제공
2. 집계 함수란 여러 행의 데이터를 바탕으로 하나의 요약된 결과 값을 계산해주는 함수
3. 집계 함수 (Aggregate Functions)
  - 여러 행의 값을 바탕으로 단일 결과 값을 계산
  - GROUP BY 절과 함께 자주 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/c9ef387f-71b3-482b-a158-3b17d19ddbce">
</div>

4. 전체 데이터 건수 파악하기 : COUNT()
  - COUNT() 함수는 특정 컬럼의 행(row) 개수를 세는 가장 기본적인 집계 함수
  - COUNT(```*```) 는 NULL 값에 상관없이 테이블의 모든 행의 개수를 셈
  - 예제) 총 주문 건수 파악하기
```sql
SELECT COUNT(*)
FROM order_stat;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/99b868ae-1c7d-4f5b-998a-568eeab618eb">
</div>

   - order_stat 테이블에는 이제 총 11개의 행이 있으므로, 총 주문 건수는 11건이라는 것을 알 수 있음
   - COUNT(```*```) 는 이처럼 테이블의 전체 규모를 파악하는 데 가장 기본적으로 사용

5. 💡 NULL과 특정 컬럼의 개수 : COUNT(컬럼) vs COUNT(```*```)
   - COUNT(```*```) : 별표(```*```)는 '모든 컬럼'을 의미
     + 더 나아가 그냥 '행 자체'를 가리킴
     + 따라서 COUNT(```*```)는 NULL 값의 존재 여부와 상관없이 테이블의 물리적인 행의 개수를 모두 셈
    
   - COUNT(컬럼명) : 특정 컬럼 이름을 지정하면, 해당 컬럼의 값 중에서 NULL 이 아닌 값의 개수만 셈
     + NULL 은 '값이 없음'을 의미하므로 개수에서 제외되는 것

   - 예제) COUNT(```*```) 와 COUNT(category) 의 차이 확인하기
```sql
SELECT
     COUNT(*) AS `전체 주문 건수`,
     COUNT(category) AS `카테고리 등록 건수`
FROM
     order_stat;
```
   - 별칭에 공백 같은 특수 문자가 필요하다면 백틱(``` ` ```)으로 감쌀 것

   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/73b3dff9-650e-49e3-88dd-2744a5bc581f">
</div>

   - COUNT(*) : NULL을 포함한 모든 행을 세어서 총 11건을 반환
   - COUNT(category) : category 컬럼에서 값이 NULL인 '홍길동'의 주문을 제외하고, 값이 존재하는 행만 세어서 10건을 반환

6. 합계와 평균 계산으로 매출 분석하기 : SUM() , AVG()
   - SUM(컬럼명) : 지정한 숫자 컬럼의 모든 값을 더하여 합계를 계산
   - AVG(컬럼명) : 지정한 숫자 컬럼의 값들의 평균을 계산
   - SUM()과 AVG() 함수는 계산 과정에서 NULL 값을 자동으로 제외 : NULL 을 0 으로 취급하지 않는다는 점을 명심
   - 예제 1) 총 매출액과 평균 주문 금액 분석하기
      + order_stat 테이블을 사용해서 우리 쇼핑몰의 총매출액과, 주문 건당 평균 구매액(객단가)을 구해보기
      + 총 매출액은 각 주문의 '상품 가격(price)'과 '주문 수량(quantity)'을 곱한 값의 합계로 계산해야 함
```sql
SELECT
     SUM(price * quantity) AS `총 매출액`,
     AVG(price * quantity) AS `평균 주문 금액`
FROM
     order_stat;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/c0d42178-4f75-429f-b721-1889aef3fc07">
</div>

   - 쿼리 한 줄로 우리 쇼핑몰의 총매출이 1,985,000원이고, 주문 한 건당 평균적으로 약 180,455원을 지출한다는 결과를 얻었음

   - 예제 2) 총 판매 상품 수량과 주문당 평균 수량 분석하기
      + 이번에는 고객들이 총 몇 개의 상품을 주문했고, 한 번 주문할 때 평균적으로 몇 개를 주문하는지 분석
      + quantity 컬럼을 사용
```sql
SELECT
     SUM(quantity) AS `총 판매 수량`,
     AVG(quantity) AS `주문당 평균 수량`
FROM
     order_stat;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/f120ee8b-cdf2-4edf-9c29-2ddd2fb33e05">
</div>

   - 지금까지 총 15개의 상품이 판매되었고, 주문 한 건당 평균 1.3636개의 상품을 구매했다는 사실을 알 수 있음
   - 이처럼 SUM과 AVG를 활용하면 데이터의 전체적인 규모와 평균적인 경향을 손쉽게 파악할 수 있음

7. 최대, 최소값으로 상품 전략 세우기 : MAX() , MIN()
    - MAX(컬럼명) : 지정한 컬럼에서 최댓값을 찾음
    - MIN(컬럼명) : 지정한 컬럼에서 최솟값을 찾음
    - 예제 1) 최고가, 최저가 상품 가격 찾기
      + order_stat 테이블에서 판매된 상품 중 가장 비싼 상품의 가격과 가장 저렴한 상품의 가격을 찾아보기
```sql
SELECT
     MAX(price) AS 최고가,
     MIN(price) AS 최저가
FROM
     order_stat;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/3ad42003-4f6a-43fa-a885-22e710d05beb">
</div>

   - 가장 비싼 상품은 450,000원, 가장 저렴한 상품은 35,000원이라는 것을 바로 알 수 있음

   - 예제 2) 최초 주문일과 최근 주문일 찾기
      + order_stat 테이블에서 가장 먼저 들어온 주문의 날짜와 가장 최근에 들어온 주문의 날짜를 찾아보기
      + order_date 컬럼에 MIN() 과 MAX() 를 적용
```sql
SELECT
     MIN(order_date) AS `최초 주문일`,
     MAX(order_date) AS `최근 주문일`
FROM
     order_stat;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/d6c75339-0f2f-498a-a5ae-a78a5cc9d123">
</div>
  
  - MAX 와 MIN 은 데이터의 범위를 파악하는 데 매우 유용

8. 💡 고유 고객 수 파악하기 : DISTINCT
   - 중복된 값을 제외하고 순수한(고유한) 값의 개수만 세고 싶을 때 사용하는 것이 바로 DISTINCT 키워드
   - DISTINCT는 집계 함수와 함께 COUNT(DISTINCT 컬럼명) 형태로 사용할 수 있음
   - 이때는 특정 컬럼에서 중복을 제거한 뒤 개수를 셈
   - 예제) 실제 구매자 수 정확히 파악하기
      + order_stat 테이블에서 중복을 포함한 고객 이름의 수와, 중복을 제거한 순수 고객 수를 함께 조회하여 그 차이를 명확하게 비교해보기
```sql
SELECT
     COUNT(customer_name) AS `총 주문 건수`,
     COUNT(DISTINCT customer_name) AS `순수 고객 수`
FROM
     order_stat;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/636839a0-dc66-4d13-8a53-d0ec0560c484">
</div>

   - COUNT(customer_name) : 중복을 포함하여 총 주문 건수인 11을 반환
   - COUNT(DISTINCT customer_name) : 고객 이름('이순신', '세종대왕', '신사임당', '장영실', '홍길동')에서 중복을 모두 제거하고 고유한 값들의 개수인 5를 반환
   - 따라서 우리 쇼핑몰에서 한음
