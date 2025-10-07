-----
### SQL 실행 순서
-----
1. 예) "고객별 총 구매 금액을 구하는데, 총 구매 금액이 40만원 이상인 고객만 보이기"
```sql
SELECT
     customer_name,
     SUM(price * quantity) AS total_purchase
FROM
     order_stat
WHERE
     total_purchase >= 400000 -- SELECT에서 만든 별칭을 사용
GROUP BY
     customer_name;
```
   - 실행 결과
```
Error Code: 1054. Unknown column 'total_purchase' in 'where clause'
```
   - 이 쿼리는 실행하면 오류가 발생 : WHERE 절에서 total_purchase라는 컬럼을 찾을 수 없다는 내용의 오류
      + 코드를 작성하는 순서와 SQL이 실제로 쿼리를 처리하는 순서(논리적 실행 순서)가 다르기 때문에 발생

2. 💡 SQL 쿼리의 논리적 실행 순서
   - 코드를 작성하는 순서는 보통 SELECT → FROM → WHERE → ... 이지만, 데이터베이스는 다음의 논리적 순서에 따라 쿼리를 해석하고 실행
   - FROM : 가장 먼저 실행되어, 어떤 테이블에서 데이터를 가져올지 결정
   - WHERE : FROM 에서 가져온 테이블의 '개별 행'을 필터링이며, GROUP BY 로 묶이기 전, 날 것 그대로의 데이터를 1차로 걸러내는 단계
   - GROUP BY : WHERE절의 필터링을 통과한 행들을 기준으로 그룹을 형성
   - HAVING : GROUP BY 를 통해 만들어진 '그룹'들을 필터링되어, 집계 함수를 이용한 조건 필터링이 여기서 이루어짐
   - SELECT : HAVING 절까지 통과한 최종 그룹들에 대해 우리가 보고자 하는 컬럼을 선택하고, SUM, COUNT 같은 집계 함수 계산, 별칭(AS) 부여 등이 모두 이 단계에서 이루어짐
   - ORDER BY : SELECT 절에서 선택된 최종 결과 후보들을 지정된 순서로 정렬
      + SELECT가 ORDER BY 보다 먼저 실행되기 때문에, ORDER BY 절에서는 SELECT 절에서 만든 별칭을 사용할 수 있음 (ORDER BY total_purchase DESC 와 같은 구문이 가능한 이유)
   - LIMIT : 정렬된 결과 중에서 최종적으로 사용자에게 반환할 행의 개수를 제한

3. 💡 논리적 실행 순서 : FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
   - 이제 WHERE 절에서 SELECT의 별칭을 쓸 수 없는 이유가 명확해짐
   - WHERE 절을 사용하는 시점에는 아직 SELECT 절이 실행되기 한참 전이라, total_purchase 같은 별칭은 세상에 존재하지도 않기 때문임

4. 참고 : SQL의 논리적인 실행 순서와 물리적인 실행 순서
   - SQL의 논리적 실행 순서에 맞추어 쿼리를 작성해야 의도한 결과를 정확히 얻을 수 있음
   - 데이터베이스 엔진은 성능 최적화를 위해 내부적으로 이 순서를 재배열해 물리적 실행 순서를 결정
   - 그러나 물리적 순서가 바뀌더라도 엔진은 동일한 결과를 보장하므로, 사용자는 논리적 실행 순서에 맞추어 쿼리를 작성하면 됨

5. 실전 예제로 따라가는 실행 순서
   - 이제 우리 쇼핑몰 데이터로 복잡한 쿼리를 하나 작성하고, 앞서 배운 7단계 실행 순서에 따라 데이터가 어떻게 가공되는지 단계별로 추적
   - 문제
     + 2025년 5월 14일 이전에 들어온 주문들 중에서(WHERE), 고객별로 그룹화하여(GROUP BY), 주문 건수가 2회 이상인 고객을 찾아서(HAVING), 해당 고객의 이름과 총 구매 금액을 조회하고(SELECT), 총 구매 금액을 기준으로 내림차순 정렬(ORDER BY)하되, 그리고 하나의 데이터만 출력
```sql
SELECT
     customer_name,
     SUM(price * quantity) AS total_purchase -- 5단계
FROM
     order_stat -- 1단계
WHERE
     order_date < '2025-05-14' -- 2단계
GROUP BY
     customer_name -- 3단계
HAVING
     COUNT(*) >= 2 -- 4단계
ORDER BY
     total_purchase DESC -- 6단계
LIMIT 1; -- 7단계
```

6. 단계별 추적
   - 1단계 : FROM order_stat
      + 우선 order_stat 테이블에 있는 11개의 모든 행을 작업대로 가져옴
   - 2단계 : WHERE order_date < '2025-05-14'
      + 가져온 11개 행을 하나씩 검사하여 order_date 가 '2025-05-14' 이전인지 확인
      + 5월 14일 이후의 주문 5건(이순신-만년필, 세종대왕-데스크, 신사임당-이어폰, 장영실-보조배터리, 홍길동USB허브)이 제외
      + 6개의 행이 다음 단계로 넘어감
```sql
SELECT *
FROM order_stat
WHERE order_date < '2025-05-14';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/911b84d2-5435-4e33-9e76-dba464befecd">
</div>

   - 3단계 : GROUP BY customer_name
      + 남아있는 6개 행을 customer_name 기준으로 묶어 그룹을 만듬
      + Group 1 (이순신) : 키보드 주문, 마우스 주문 (2개 행)
      + Group 2 (세종대왕) : SQL책 주문, 모니터 주문 (2개 행)
      + Group 3 (신사임당) : 의자 주문 (1개 행)
      + Group 4 (장영실) : 파이썬책 주문 (1개 행)
      + 총 4개의 그룹이 생성

   - 4단계 : HAVING COUNT(```*```) >= 2
      + 생성된 4개의 그룹을 대상으로, 각 그룹의 행 개수(COUNT(```*```))가 2 이상인지 검사
      + Group 1 (이순신) : COUNT(```*```)는 2. 조건 일치, 통과
      + Group 2 (세종대왕) : COUNT(```*```)는 2. 조건 일치, 통과
      + Group 3 (신사임당) : COUNT(```*```)는 1. 조건 불일치, 제외
      + Group 4 (장영실):COUNT(```*```)는 1. 조건 불일치, 제외
      + 오직 '이순신'과 '세종대왕' 그룹 두 개만이 살아남았음

   - 5단계 : SELECT customer_name, SUM(...) AS total_purchase
      + 최종적으로 살아남은 두 그룹에 대해 SELECT 절을 실행
      + '이순신' 그룹 : SUM 계산: 150000 * 1 + 80000 * 1 = 230000
      + '세종대왕' 그룹 : SUM 계산: 35000 * 2 + 450000 * 1 = 520000
      + 이제 최종 결과 집합의 후보로 다음 두 행이 만들어짐
<div align="center">
<img src="https://github.com/user-attachments/assets/64d82f95-3339-484f-af1f-aa7c3877c210">
</div>

   - 6단계 : ORDER BY total_purchase DESC
    + SELECT 된 두 행을 total_purchase 기준으로 내림차순 정렬
    + 520,000이 230,000보다 크므로 '세종대왕' 행이 먼저 옴
<div align="center">
<img src="https://github.com/user-attachments/assets/9465d0ea-84fa-43b5-97bd-189f0de242dd">
</div>

   - 7단계 : LIMIT 1, 최종 결과 (하나의 행만 선택)
<div align="center">
<img src="https://github.com/user-attachments/assets/c15e227b-816e-4cb7-82cb-efc51cb46a5a">
</div>
