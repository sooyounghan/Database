-----
### HAVING - 그룹 필터링 1
-----
```sql
SELECT
     category,
     SUM(price * quantity) AS total_sales
FROM
     order_stat
GROUP BY
     category;
```
1. 이 쿼리는 order_stat 테이블의 모든 주문 기록을 category 기준으로 묶고, 각 그룹(카테고리)별로 price * quantity 의 합계를 계산
2. 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/5e805289-fac2-4c4d-a36e-553cf88306f0">
</div>

   - 이런 요구사항을 WHERE SUM(price * quantity) >= 500000 와 같이 설정이 가능한가?
```sql
SELECT
     category,
     SUM(price * quantity) AS total_sales
FROM
     order_stat
WHERE
    SUM(price * quantity) >= 500000
GROUP BY
     category;
```
   - 실행 결과
```
Error Code: 1111. Invalid use of group function
```
   - 오류가 발생
      + 💡 이유는 WHERE 절은 그룹화가 이루어지기 전, 즉 테이블의 개별 행 하나하나에 대해 조건을 검사하기 때문임
      + SUM()과 같은 집계 함수는 GROUP BY를 통해 여러 행이 하나의 그룹으로 묶인 후에야 계산될 수 있는 값
      + 따라서 WHERE 절의 입장에서는 아직 존재하지도 않는 값을 조건으로 사용하려는 셈이니, 오류를 내뱉는 것

3. 💡 SQL 작동 순서 : FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
   - 그룹으로 묶은 결과에 대해 다시 필터링을 하고 싶을 때, SQL은 HAVING 이라는 도구를 제공
   - WHERE, HAVING 개념 비유
      + 전체 학생(테이블 데이터)들이 운동장에 모여있음
      + 선생님이 "안경 쓴 학생들만 앞으로 나와!"라고 외침 (WHERE : 개별 행 필터링)
      + 앞으로 나온 학생들을 대상으로 "같은 반 학생들끼리 모여!"라고 함 (GROUP BY : 그룹화)
      + 반별로 모인 그룹들을 보며 선생님이 마지막으로 "이 중에서, 반 평균 점수가 80점 이상인 그룹만 남아!"라고 외침 (HAVING : 그룹 필터링)
   - WHERE는 그룹화 이전에 개개인을 걸러내는 조건이고, HAVING은 그룹화 이후에 그룹 자체를 걸러내는 조건

4. 그룹을 위한 필터 : HAVING 절의 사용법
   - HAVING 절은 GROUP BY 절 바로 뒤에 위치
   - 그룹화된 결과에 대한 조건을 지정하는 역할
     + WHERE 가 원본 데이터의 개별 행을 위한 필터라면, HAVING은 GROUP BY를 통과한 그룹을 위한 필터
   - 💡 개념 설명 : HAVING 절의 조건문에는 SUM() , COUNT() , AVG() 같은 집계 함수를 사용할 수 있음
     + GROUP BY 를 통해 생성된 여러 그룹들 중에서 HAVING 절의 조건을 만족하는 그룹들만 최종 결과에 포함
   - 💡 작동 순서: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY 순서로 동작 : GROUP BY로 그룹을 만들고, HAVING 으로 그 그룹들을 걸러냄

5. 예제 1) '핵심 카테고리' 필터링하기
   - 1단계 : 먼저 카테고리별로 그룹화하여 총 매출액 집계하기
      + HAVING을 사용하기 전에, 우리는 먼저 GROUP BY를 사용해서 각 카테고리별로 총 얼마의 매출이 발생했는지 계산
```sql
SELECT
     category,
     SUM(price * quantity) AS total_sales
FROM
     order_stat
GROUP BY
     category;
```
   - 이 쿼리는 order_stat 테이블의 모든 주문 기록을 category기준으로 묶고, 각 그룹(카테고리)별로 price * quantity 의 합계를 계산
   - 1단계 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/26fe778d-38b9-40c3-a56d-ea466dee267c">
</div>

   - 결과를 보면 각 카테고리의 총 매출액이 잘 계산되었음
   - 이제 이 결과 중에서 '총 매출액 50만 원 이상'이라는 조건을 만족하는 그룹만 걸러내면 됨
   - 바로 이럴 때 HAVING이 등장

   - 2단계 : HAVING 절로 2차 필터링
      + 이제 1단계 쿼리에 HAVING 절을 추가해서 '총 매출액이 50만 원 이상'인 그룹만 남겨보기
```sql
SELECT
     category,
     SUM(price * quantity) AS total_sales
FROM
     order_stat
GROUP BY
     category
HAVING
     SUM(price * quantity) >= 500000;
```
   - 2단계 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/9e6dcf36-e00d-425a-8673-84ec0131b209">
</div>

   - GROUP BY의 결과로 나왔던 5개의 카테고리 그룹 중에서, 총 매출액이 50만 원 미만이었던 '도서', '문구', 그리고 NULL 카테고리는 HAVING 절의 SUM(...) >= 500000 조건을 만족하지 못해 필터링되어 제외
   - 이처럼 HAVING은 GROUP BY의 결과에 대한 필터링 조건을 적용

6. HAVING에서 별칭 사용
   - HAVING에 SUM(price * quantity) 대신에 SELECT 절에서 사용한 total_sales 별칭을 대신 사용
```sql
SELECT
     category,
     SUM(price * quantity) AS total_sales
FROM
     order_stat
GROUP BY
     category
HAVING
     total_sales >= 500000;
```
   - MySQL에서는 잘 작동
   - SQL 표준과 HAVING에서 별칭 사용
     + SQL 표준 : 표준 SQL 문법에 따르면 HAVING 절은 SELECT 절보다 먼저 처리
     + 따라서 SELECT 절에서 지정한 별칭(alias)을 HAVING 절에서 사용하는 것은 원칙적으로 불가능
   - 💡 DBMS별 차이 :
     + MySQL, PostgreSQL 등 : 사용자의 편의를 위해 표준을 확장하여 HAVING 절에서 SELECT의 별칭을 사용할 수 있도록 허용
     + SQL Server, Oracle 등 : SQL 표준을 비교적 엄격하게 준수하므로 HAVING 절에서 별칭을 사용하면 오류 발생
   - 💡 결론 : 여러 데이터베이스 환경에서 코드가 문제없이 작동하도록 하려면, 설명한 대로 HAVING 절에 SUM(price * quantity)와 같이 집계 함수 표현식을 직접 사용하는 것이 안전하고 호환성이 높은 방법

7. 예제 2) 충성 고객 필터링하기 : '3회 이상 주문한' 충성 고객을 찾아보기
   - 1단계 : 고객별 총 주문 횟수 집계하기 : 먼저 각 고객이 몇 번이나 주문했는지 COUNT(*) 를 사용해서 계산해보기
```sql
SELECT
     customer_name,
     COUNT(*) AS order_count
FROM
     order_stat
GROUP BY
     customer_name;
```
  - 1단계 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/f7e938dd-d98c-43fe-b6e6-f2d95c1ea990">
</div>

   - 고객별로 총 몇 번의 주문(행)이 있었는지 정확히 집계
   - 이제 여기서 주문 횟수가 3회 이상인 고객만 남기면 됨


   - 2단계 : HAVING 절을 추가하여 주문 횟수 3회 이상인 그룹 필터링하기
     + 1단계 결과에 HAVING COUNT(*) >= 3 이라는 조건을 추가하여 우리가 원하는 충성 고객 목록을 완성해보기
```sql
SELECT
     customer_name,
     COUNT(*) AS order_count
FROM
     order_stat
GROUP BY
     customer_name
HAVING
     COUNT(*) >= 3;
```
  - 2단계 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/c73fa560-ac82-4ed4-9d3e-83f57852cb6b">
</div>

   - 총 주문 횟수가 3회 미만이었던 신사임당, 장영실, 홍길동 그룹은 HAVING 절의 조건을 통과하지 못하고, 3회씩 주문한 이순신과 세종대왕만이 최종 결과에 포함
   - 이처럼 HAVING을 사용하면 그룹화된 데이터에 대해 원하는 조건으로 정밀하게 필터링할 수 있음
