-----
### SELECT문
-----
1. 데이터베이스에 저장된 데이터를 가져오는 명령문
2. 상황이나 조건에 맞는 데이터를 추출 가능
3. 기본 구조

```sql
SELECT {* | [테이블명.]컬럼명 1, .. 컬럼명 [AS 별칭]} / 수식 또는 함수도 가능
FROM [데이터베이스명.]테이블명
[WHERE 조건]
[GROUP BY 그룹기준]
[HAVING 그룹조건]
[ORDER BY 정렬 기준 정렬 방법];
```

4. 예제
  A. 사원의 모든 정보를 가져옴 (사원정보 : employees 테이블)
```sql
SELECT *
FROM EMPLOYEES;
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/e0d404da-a9db-4d56-84c8-0c87e925dfea">
</div>

  - DESC EMPOLYEES; 를 통해 나오는 컬럼의 순서대로 출력 (즉, 테이블에 선언한 컬럼 순서대로 등장)

  B. 부서 정보를 모두 가져옴
```sql
SELECT *
FROM DEPARTMENTS;
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/1f2b4281-12c6-4604-8629-11d20a4e4659">
</div>

  C. 부서 관리자 정보를 모두 가져옴
```sql
SELECT *
FROM DEPT_MANAGER;
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/1f861f5d-1424-4c3e-93e1-878600651581">
</div>

  D. 사원들 직함 정보를 모두 가져옴
```sql
SELECT *
FROM  TITLES;
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/dfbb3fa7-ee04-4d73-a8aa-e8ed3d2ca3bc">
</div>

-----
### SELECT문 - 특정 Column 가져오기
-----
```sql
SELECT 컬럼명1, 컬럼명2, 컬럼명3
FROM 테이블명;
```
1. 사원의 정보 중 사원번호, 사원 이름을 가져옴
```sql
SELECT EMP_NO, FIRST_NAME, LAST_NAME
FROM EMPLOYEES;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/08df84cc-e637-4eab-a690-cbecb2a6005f">
</div>

2. 사원의 사원번호, 생년월일, 성별을 가져옴
```sql
SELECT EMP_NO, BIRTH_DATE, GENDER
FROM EMPLOYEES;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/7ab39f2a-84cd-4a49-86a8-067273eb1b9d">
</div>

3. 부서의 부서번호, 부서 이름을 가져옴
```sql
SELECT DEPT_NO, DEPT_NAME
FROM DEPARTMENTS;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/da80358c-e445-433b-b8ca-1e2d9158fa81">
</div>

4. 각 사원의 사원번호, 급여액을 가져옴
```sql
SELECT EMP_NO, SALARY 
FROM SALARIES;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/0430e411-a4ff-4d79-84af-cf3c8403f25c">
</div>

5. 각 사원의 사원번호, 직함 이름을 가져옴
```sql
SELECT EMP_NO, TITLE
FROM TITLES;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/542b134a-ed2b-4ba7-a52d-6866b410272d">
</div>
