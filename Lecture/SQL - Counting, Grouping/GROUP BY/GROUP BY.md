-----
### GROUP BY - 그룹으로 묶기
-----
1. 그룹화의 첫걸음 : GROUP BY 기본
   - GROUP BY는 이름 그대로, 특정 컬럼의 값이 같은 행들을 하나의 그룹으로 묶어주는 역할을 함
   - 개념 설명 : GROUP BY 절에 그룹화의 기준이 될 컬럼을 지정
     + 이렇게 그룹화된 결과에 집계 함수를 적용하면, 전체에 대한 통계가 아닌 '각 그룹에 대한 통계'를 낼 수 있음

2. 예제) 카테고리별 주문 건수
   - order_stat 테이블의 데이터를 category컬럼을 기준으로 그룹으로 묶음 : '전자기기'는 '전자기기'끼리, '도서'는 '도서'끼리 묶는 것
   - 그런 다음 각 그룹에 COUNT(*) 를 적용하면, 각 카테고리 그룹에 속한 주문(행)의 개수를 셀 수 있음
```sql
SELECT
     category,
     COUNT(*) AS `카테고리별 주문 건수`
FROM
     order_stat
GROUP BY
     category;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/b59bdd72-9b0d-409d-ba34-6b1043226411">
</div>

   - 결과 분석
     + '전자기기' 카테고리에서 5건의 주문이 발생하여 가장 인기가 많다는 것을 한눈에 알 수 있dma
     + GROUP BY category 는 전체 데이터를 카테고리 값에 따라 그룹으로 나누고, COUNT(```*```) 는 각 그룹에 몇 개의 주문이 있는지 계산
     + 💡 여기서 주목할 점은 category가 NULL인 데이터도 하나의 그룹으로 묶여서 결과에 포함되었다는 것
     + 💡 GROUP BY 는 NULL 값 또한 하나의 독립된 그룹으로 취급하여 집계
     + 실무에서는 이런 NULL 그룹을 통해 데이터가 누락되었음을 발견하고 데이터 정제의 필요성을 인지할 수 있음

3. 예제) 고객별로 총 몇 번이나 주문했을까?
   - order_stat 테이블에는 한 고객이 여러 번 주문한 기록이 섞여 있음
   - GROUP BY 를 사용해서 customer_name이 같은 행들을 하나의 그룹으로 묶어볼 것
   - 그리고 각 그룹에 대해 COUNT(*) 를 실행하면 '고객별 주문 횟수'를 구할 수 있음
```sql
SELECT
     customer_name,
     COUNT(*) AS `주문 횟수`
FROM
     order_stat
GROUP BY
     customer_name;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/6926c766-55f4-4b3a-95e7-853aa704aac4">
</div>

   - GROUP BY customer_name은 order_stat 테이블의 11개 행을 고객 이름에 따라 5개의 그룹으로 묶음
   - 그리고 SELECT 절의 COUNT(*)는 각 그룹에 속한 행의 개수를 세어 반환
   - 이로써 우리는 각 고객이 몇 번씩 주문했는지 명확하게 알 수 있음

4. 그룹별로 심층 분석하기 : GROUP BY 와 집계 함수
   - GROUP BY의 진정한 강력함은 다양한 집계 함수와 함께 사용될 때 발휘
   - COUNT() 뿐만 아니라 SUM(), AVG(), MAX(), MIN() 등을 모두 적용할 수 있음
   - 예제 시나리오) 고객별 구매 활동 분석 (VIP 고객 찾기)
      + 이번에는 한 걸음 더 나아가, 고객별로 총 주문 횟수, 총 구매 수량, 그리고 총 구매 금액을 함께 계산
      + 이를 통해 어떤 고객이 우리 쇼핑몰의 진정한 VIP인지 찾아낼 수 있음
```sql
SELECT
     customer_name,
     COUNT(*) AS `총 주문 횟수`,
     SUM(quantity) AS `총 구매 수량`,
     SUM(price * quantity) AS `총 구매 금액`
FROM
     order_stat
GROUP BY
     customer_name
ORDER BY
     `총 구매 금액` DESC; -- 백틱 사용 주의!
```
   - ORDER BY 를 추가하여 총 구매 금액이 높은 순으로 정렬하면 VIP 고객을 한눈에 파악하기 좋음
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/8d99c0c6-3c50-45eb-98d0-55fdac644068">
</div>

   - 이 결과를 보면 세종대왕이 총 840,000원을 사용하여 우리 쇼핑몰의 최고 VIP 고객임을 명확히 알 수 있음
   - 이처럼 GROUP BY와 집계 함수를 조합하면 강력한 비즈니스 분석이 가능

5. 💡 주의) ORDER BY 백틱 사용
   - ORDER BY 에서 총 구매 금액 문자에 백틱(`)을 사용 : 이렇게 백틱을 사용해야 SELECT에서 사용한 컬럼명으로 인식
   - MySQL에서 작은 따옴표('), 큰 따옴표(")를 사용하면 문자 상수로 인식 : 따라서 SELECT에서 사용한 컬럼명으로 인식하지 못함
   - 이렇게 되면 모두 같은 문자 상수로 정렬되어 버리므로, 여기서는 모든 컬럼이 "총 구매 금액"이라는 문자 값으로 정렬되므로 정렬 효과가 없음
   - 따라서 정렬이 안되는 문제가 발생
   - 작은 따옴표를 사용하는 잘못된 예시
```sql
SELECT
     customer_name,
     COUNT(*) AS `총 주문 횟수`,
     SUM(quantity) AS `총 구매 수량`,
     SUM(price * quantity) AS `총 구매 금액`,
     '총 구매 금액' AS `정렬 값`
FROM
     order_stat
GROUP BY
     customer_name
ORDER BY
     '총 구매 금액' DESC; -- 작은 따옴표('), 문자로 인식
```
   - 이렇게 되면 모든 컬럼이 '총 구매 금액'이라는 똑같은 문자 값으로 정렬
   - 따라서 정렬이 정상적으로 이루어지지 않음
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/fd58e170-65f8-4656-8396-354f9233e66f">
</div>

   - 똑같은 '총 구매 금액'이라는 문자를 기준으로 정렬되며, 결과적으로 총 구매 금액으로 정렬되지 않음

6. 더 세분화된 그룹으로 분석하기 : 여러 컬럼 기준 그룹화
   - GROUP BY 절에 여러 컬럼을 콤마(,)로 구분하여 나열하면, 다중 기준으로 데이터를 그룹화할 수 있음
   - GROUP BY 컬럼1, 컬럼2 : 컬럼1로 먼저 그룹화하고, 그 안에서 다시 컬럼2로 그룹화하여 더 세분화된 그룹을 만듬
   - 예제 시나리오) 고객이 어떤 카테고리에서 얼마를 사용했는가?
      + '어떤 고객이(customer_name)', '어떤 상품 카테고리에서(category)' 얼마를 썼는지 그룹으로 묶어보기
```sql
SELECT
     customer_name,
     category,
     SUM(price * quantity) AS `카테고리별 구매 금액`
FROM
     order_stat
GROUP BY
     customer_name, category
ORDER BY
     customer_name, `카테고리별 구매 금액` DESC;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/5bb8666c-4cc5-40b0-b8ba-08aa008e78d9">
</div>

   - 이 결과는 매우 구체적인 정보를 제공
     + 예를 들어 세종대왕이 전자기기 카테고리에서 구매한 금액의 합, 가구에서 구매한 금액의 합, 도서에서 구매한 합을 각각 따로 구할 수 있음
