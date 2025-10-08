-----
### 셀프 조인
-----
1. 연결해야 할 대상이 다른 테이블이 아니라 바로 '자기 자신'이라면 어떻게 해야 할까?
   - 각 직원의 이름과 바로 위 직속 상사의 이름을 나란히 함께 출력하려면 어떻게 해야 할까?
   - employees 테이블의 구조
<div align="center">
<img src="https://github.com/user-attachments/assets/079c9a28-c924-4524-a458-8afc5b7ecde1">
</div>

```sql
SELECT * FROM employees;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/285dbd5b-3eb0-43c5-b45b-ea0d80a08c1b">
</div>

   - '홍사원'(employee_id:6)의 manager_id:4
     + 이 상사의 이름을 알려면, 다시 이 테이블에서 id가 4인 직원을 찾아야 하며, '최과장'이라는 것을 알 수 있음
   - 이처럼 한 테이블 안에서 자신의 컬럼(manager_id)이 같은 테이블의 다른 컬럼(employee_id)을 참조하는 구조 : '자기 참조 관계'
   - 이런 구조의 데이터를 한 번의 쿼리로 조회하기 위해 사용하는 기술이 바로 SELF JOIN

2. SELF JOIN의 개념과 원리
   - SELF JOIN은 INNER JOIN , OUTER JOIN 처럼 새로운 종류의 JOIN 명령어가 아님
   - 이것은 하나의 테이블을 자기 자신과 조인하는 '기법'을 일컫는 말
   - SQL이 이 기법을 가능하게 하는 핵심 원리는 바로 테이블 별칭(Alias)
     + 하나의 테이블에 서로 다른 별칭을 두 개 부여함으로써, 데이터베이스가 이들을 마치 다른 두 개의 테이블인 것처럼 인식하게 만드는 것
     + employees 테이블을 두 개 복사해서 하나는 직원을 나타내는 e(employee)로, 다른 하나는 상사를 나타내는 m(manager)으로 사용한다고 상상하면 이해하기 쉬움
        * e 테이블 : 모든 직원의 목록
        * m 테이블 : 모든 상사의 목록 (실체는 똑같은 employees 테이블)
     + 그리고 e 테이블의 manager_id와 m 테이블의 employee_id가 같은 것들을 연결(JOIN)해주면, 직원의 이름(e.name)과 상사의 이름(m.name)을 한 줄에 나란히 놓을 수 있게 됨

3. 실습 : 직원-상사 목록 만들기
   - 1단계 : INNER JOIN을 이용한 SELF JOIN
     + 가장 기본적인 INNER JOIN 으로 두 가상 테이블 e와 m을 연결
     + 연결 조건은 "직원의 매니저 ID(e.manager_id)가 상사의 직원 ID(m.employee_id)와 같을 때"
```sql
SELECT
   e.name AS employee_name,
   m.name AS manager_name
FROM
   employees e
JOIN
   employees m
ON
   e.manager_id = m.employee_id;
```
   - e.name은 직원의 이름, m.name은 상사의 이름
   - 컬럼에도 AS를 사용해 별칭을 붙여주면 결과를 이해하기가 훨씬 수월 (물론 AS는 생략할 수 있음)
<div align="center">
<img src="https://github.com/user-attachments/assets/8d337b74-8453-4631-947b-f7a3830da005">
</div>

   - 결과를 보면, 각 직원 옆에 직속 상사의 이름이 정확히 출력된 것을 확인할 수 있음
   - 한 가지 이상한 점 : 전체 직원 중 '김회장'이 employee_name 목록에 보이지 않음
     + 그의 manager_id는 NULL 이기 때문임 : INNER JOIN은 ON 조건이 맞는, 즉 e.manager_id에 값이 있는 데이터만 결과에 포함시키므로 manager_id가 NULL인 '김회장'은 조인 대상에서 제외된 것

   - 2단계 : LEFT JOIN 을 이용한 SELF JOIN
     + 만약 상사가 없는 최상위 리더, 즉 '김회장'까지 포함한 전체 직원 목록을 보고 싶다면 어떻게 해야 할까?
     + 이럴 때 바로 LEFT JOIN이 필요 : 직원을 나타내는 e 테이블을 기준으로 삼고, 상사 정보를 왼쪽에 붙이는 것
     + 상사 정보가 없으면(manager_id 가 NULL 이면) 그 자리는 NULL 로 표시될 것
```sql
SELECT
   e.name AS employee_name,
   m.name AS manager_name
FROM
   employees AS e
LEFT JOIN
   employees AS m
ON
   e.manager_id = m.employee_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/64a79ef7-add7-4e0a-9698-67f08314a201">
</div>

4. 이처럼 SELF JOIN 은 테이블 별칭을 활용하여 자기 참조 관계를 풀어내는 유용한 기법
   - 조직도뿐만 아니라 웹사이트의 카테고리와 서브카테고리, 게시판의 원본 글과 답변 글 같은 계층형 데이터를 다룰 때 반드시 필요한 기술
