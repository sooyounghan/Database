-----
### 일대다(1:N) 다대일(N:1) 관계 - 조인
-----
1. 💡 일대다와 다대일은 같은 관계지만, 어떤 테이블을 기준으로 조인(JOIN)하느냐에 따라 결과의 형태가 완전히 달라짐
   - 조인을 했을 때 기준 테이블의 행 수보다 결과 행의 수가 더 늘어날 수 있는데 이것을 데이터 뻥튀기라고 함
   - '데이터 뻥튀기' 현상을 이해하는 것은 SQL 쿼리 작성의 핵심

2. 다대일(N:1) 조인 : 데이터가 뻥튀기되지 않음
   - '다(N)' 쪽인 member 테이블을 기준으로 '일(1)' 쪽인 team 테이블을 조인
   - 모든 회원의 이름과 그들이 속한 팀의 이름을 조회하는 쿼리
   - 먼저 member 테이블만 조회해서 기준이 되는 데이터의 수를 확인
```sql
SELECT *
FROM member;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/167b9ec6-a24c-4c78-af6d-e92bdf7f9c07">
</div>

   - 총 4개의 행이 존재 : 이제 조인을 사용해서 이 member 테이블에 team 정보 붙이기
```sql
SELECT
   m.name AS member_name,
   t.name AS team_name
FROM
   member m
LEFT JOIN
   team t ON m.team_id = t.team_id;
```
   - '이기획'처럼 team 이 없는 회원도 확인하려면 LEFT JOIN을 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/6a501504-6b85-45bf-bc44-86c133dcdabc">
</div>

   - 결과를 보면, 조인 전 member 테이블의 행 수(4개)와 정확히 일치하는 4개의 행이 출력
   - 각 회원은 단 하나의 팀에만 속하기 때문에, member 를 기준으로 team 정보를 가져와도 결과 행의 수가 늘어나지 않음
   - 매우 예측 가능하고 안정적인 조인

3. 일대다(1:N) 조인 : 데이터가 뻥튀기
   - 이번에는 반대로, '일(1)' 쪽인 team 테이블을 기준으로 '다(N)' 쪽인 member 테이블을 조인
   - 먼저 기준이 되는 team 테이블의 데이터를 확인
```sql
SELECT *
FROM team;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/7529a4ca-1834-4149-a90a-cf79ca33444a">
</div>

   - 총 2개의 행이 존재 : 이제 여기에 소속된 member 정보를 붙여보기
```sql
SELECT
   t.name AS team_name,
   m.name AS member_name
FROM
   team t
JOIN
   member m ON t.team_id = m.team_id;
```
   - 소속 팀이 없는 '이기획' 회원은 제외 : 참고로 이 쿼리에 RIGHT JOIN을 사용하면 이기획 회원도 조회할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/fd46c2e8-9655-412f-8866-9f7c4df24731">
</div>

   - 분명 team 테이블에는 행이 2개뿐이었지만, 조인 결과는 3개의 행이 나옴
     + '개발팀'은 2명의 팀원을 가지고 있으므로, 조인 과정에서 '개발팀' 행이 2개로 복제되어 각 팀원과 연결
     + 이것이 바로 '1' 쪽에서 'N' 쪽을 조인할 때 발생하는 데이터 뻥튀기 현상

   - 💡 이 현상 자체는 오류가 아님
     + 오히려 이런 특징을 활용하여 '특정 팀에 속한 모든 팀원 목록' 같은 기능을 구현하는 것
     + 다만, GROUP BY 나 COUNT 같은 집계 함수를 사용할 때 이 데이터 뻥튀기를 인지하지 못하면 잘못된 결과를 얻을 수 있으므로 반드시 이해하고 넘어가야 함

4. 조인과 뻥튀기 간단 정리
   - 다대일 조인 : FK → PK로 조인
      + 기준 테이블의 데이터보다 더 많이 조회되는 뻥튀기 현상이 발생하지 않음
   - 일대다 조인 : PK → FK로 조인
     + 기준 테이블의 데이터보다 더 많이 조회되는 뻥튀기 현상이 발생할 수 있음
