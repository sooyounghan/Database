-----
### 문제와 풀이
-----
1. 문제 1 : 기본 통계 조회하기
   - order_stat 테이블을 사용하여 쇼핑몰의 전체 주문 건수와, 카테고리 정보가 누락되지 않은(NULL이 아닌) 주문 건수를 각각 조회
   - 컬럼의 별칭은 각각 '총 주문 건수', '카테고리 보유 건수'로 지정
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/622b63f2-bd3c-4ffd-9f86-fa2aacbc1ee2">
</div>

   - 정답
```sql
SELECT
     COUNT(*) AS `총 주문 건수`,
     COUNT(category) AS `카테고리 보유 건수`
FROM
     order_stat;
```

2. 문제 2 : 쇼핑몰 매출 현황 파악하기
   - order_stat 테이블을 사용하여 우리 쇼핑몰의 총 매출액, 평균 주문 금액(주문 1건당 매출액), 판매된 상품들의 최고 단가와 최저 단가를 한 번에 조회
     + 총 매출액 : price * quantity 의 합계
     + 평균 주문 금액 : price * quantity 의 평균
     + 최고 단가 : price 의 최댓값
     + 최저 단가 : price 의 최솟값
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0db8b709-8522-454f-b65b-17473942184e">
</div>

   - 정답
```sql
SELECT
     SUM(price * quantity) AS `총 매출액`,
     AVG(price * quantity) AS `평균 주문 금액`,
     MAX(price) AS `최고 단가`,
     MIN(price) AS `최저 단가`
FROM
     order_stat;
```

3. 문제 3 : 카테고리별 실적 분석하기
   - order_stat 테이블을 category별로 그룹화하여, 각 카테고리별로 총 판매된 상품 수량(quantity의 합계)과 총 매출액(price * quantity 의 합계)을 계산
   - 결과는 총 매출액이 높은 순서대로 정렬
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/404676f4-40ed-48c9-8df9-916e7343d239">
</div>

   - 정답
```sql
SELECT
     category,
     SUM(quantity) AS `카테고리별 총 판매 수량`,
     SUM(price * quantity) AS `카테고리별 총 매출액`
FROM
     order_stat
GROUP BY
     category
ORDER BY
     `카테고리별 총 매출액` DESC;
```
   - 💡 ORDER BY에서 SELECT 절의 별칭 컬럼을 선택할 때는 백틱(`)을 사용

4. 문제 4 : 고객별 주문 통계 분석하기
   - order_stat 테이블을 사용하여 고객별로 총 주문 횟수와 총 구매한 상품의 수량(quantity)을 계산
   - 결과는 주문 횟수가 많은 순으로, 주문 횟수가 같다면 총 구매 수량이 많은 순으로 정렬
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/c8418548-776a-48cf-8005-d25b5213821e">
</div>

   - 정답
```sql
SELECT
     customer_name,
     COUNT(*) AS `총 주문 횟수`,
     SUM(quantity) AS `총 구매 수량`
FROM
     order_stat
GROUP BY
     customer_name
ORDER BY
     `총 주문 횟수` DESC, `총 구매 수량` DESC;
```

5. 문제 5 : VIP 고객 필터링하기
   - order_stat 테이블에서 고객별 총 구매 금액을 계산하고, 총 구매 금액이 40만 원 이상인 'VIP 고객' 목록만 조회
   - 결과에는 고객 이름과 총 구매 금액이 포함되어야 하며, 총 구매 금액이 높은 순으로 정렬
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/a2798c11-3855-43b7-84bb-760cb45cdbc0">
</div>

   - 정답
```sql
SELECT
     customer_name,
     SUM(price * quantity) AS `총 구매 금액`
FROM
     order_stat
GROUP BY
     customer_name
HAVING
     SUM(price * quantity) >= 400000
ORDER BY
     `총 구매 금액` DESC;
```

6. 💡 문제 6 : 복합 조건으로 핵심 고객 그룹 찾기
   - order_stat 테이블에서 '도서' 카테고리를 제외한(WHERE) 주문들 중에서, 2회 이상 주문한 고객들을 찾아(GROUPBY, HAVING), 그 고객들의 이름, 주문 횟수, 총 사용 금액을 조회
   - 결과는 총 사용 금액이 높은 순으로 정렬
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/a1fe7442-37be-4359-9e2e-2833112722b7">
</div>

   - 정답
```sql
SELECT
     customer_name,
     COUNT(*) AS `주문 횟수`,
     SUM(price * quantity) AS `총 사용 금액`
FROM
     order_stat
WHERE
     category != '도서' OR category IS NULL -- 💡 '도서' 카테고리가 아닌 주문, NULL도 포함 (NOT IN ('도서') X)
GROUP BY
     customer_name
HAVING
     COUNT(*) >= 2
ORDER BY
     `총 사용 금액` DESC;
 ```
