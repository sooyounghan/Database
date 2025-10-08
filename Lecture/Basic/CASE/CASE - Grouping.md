-----
### CASE 문 - 그룹핑
-----
1. 데이터 분류 및 그룹핑
   - 고객들을 출생 연대에 따라 '1990년대생', '1980년대생', '그 이전 출생'으로 분류하고, 각 그룹에 고객이 총 몇 명씩 있는지 알고 싶은 경우
   - 이 문제를 해결하기 위한 전략은 두 단계로 나뉨
     + 분류(Classification) : CASE 문을 사용해 각 고객에게 '1990년대생', '1980년대생', '그 이전 출생' 중 하나의 라벨을 붙여줌
     + 집계(Aggregation) : 1단계에서 만들어진 라벨을 기준으로 GROUP BY하고, COUNT() 함수를 사용해 각 라벨(그룹)에 속한 고객 수를 셈

2. 단계별 쿼리 작성
   - 먼저, 1단계인 '분류' 작업부터 쿼리로 작성 : users 테이블의 birth_date 컬럼에서 YEAR() 함수로 연도만 추출하여 조건을 만듬
   - 1단계 : CASE 문으로 각 고객 분류하기
```sql
SELECT
   name,
   birth_date,
   CASE
       WHEN YEAR(birth_date) >= 1990 THEN '1990년대생'
       WHEN YEAR(birth_date) >= 1980 THEN '1980년대생'
       ELSE '그 이전 출생'
   END AS birth_decade
FROM
   users;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c16746c9-04ed-4f79-9382-3cfdfe170eed">
</div>

   - 이렇게 CASE 문이 만들어낸 새로운 가상 컬럼 birth_decade가 성공적으로 생성된 것을 볼 수 있음
   - 2단계 : GROUP BY로 그룹화하고 COUNT로 집계하기
     + 가상 컬럼 birth_decade를 GROUP BY의 기준으로 삼으면 됨
```sql
SELECT
     CASE
         WHEN YEAR(birth_date) >= 1990 THEN '1990년대생'
         WHEN YEAR(birth_date) >= 1980 THEN '1980년대생'
         ELSE '그 이전 출생'
     END AS birth_decade,
     COUNT(*) AS customer_count
FROM
     users
GROUP BY
     CASE
         WHEN YEAR(birth_date) >= 1990 THEN '1990년대생'
         WHEN YEAR(birth_date) >= 1980 THEN '1980년대생'
         ELSE '그 이전 출생'
     END;
```
<div align="center">
<img src=""https://github.com/user-attachments/assets/eb2c618b-d74c-4ff7-be0e-3d41d7d2b8c4">
</div>

   - SQL 표준의 논리적 실행 순서에 따르면 GROUP BY 절이 SELECT 절보다 먼저 처리
   - 따라서 원칙적으로는 SELECT 절에서 정의한 별칭(birth_decade)을 GROUP BY 절에서 사용할 수 없음
   - 💡 하지만 MySQL을 포함한 최신 버전의 많은 데이터베이스들은 사용자 편의를 위해 이러한 별칭 사용을 예외적으로 허용
```sql
SELECT
     CASE
         WHEN YEAR(birth_date) >= 1990 THEN '1990년대생'
         WHEN YEAR(birth_date) >= 1980 THEN '1980년대생'
         ELSE '그 이전 출생'
     END AS birth_decade,
     COUNT(*) AS customer_count
FROM
     users
GROUP BY
     birth_decade;
```
   - GROUP BY 절에 CASE 문 전체를 다시 쓸 필요 없이, SELECT 절에서 AS로 지정한 별칭(birth_decade)을 바로 사용 : 이는 쿼리를 훨씬 간결하고 읽기 쉽게 만들어줌

3. 이처럼 CASE 문을 이용해 기존에 없던 새로운 분류 기준을 동적으로 생성하고, 이를 GROUP BY와 결합하는 패턴은 실무 데이터 분석과 리포팅에서 정말로 흔하게 사용되는 핵심 기술 중 하나
   - CASE 문은 이렇게 그룹의 '기준'을 만드는 역할도 하지만, 집계 함수인 COUNT나 SUM의 '내부'로 들어가서 특정 조건에 맞는 값만 골라서 집계하는 훨씬 더 정교한 역할도 수행할 수 있음
