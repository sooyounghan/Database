-----
### 정렬
-----
1. 데이터를 가져올 때 오름차순 혹은 내림차순으로 정렬하여 가져올 수 있음
```sql
SELECT [테이블명.]컬럼명1 ...
FROM [데이터베이스명.]테이블명
[WHERE [조건]]
ORDER BY 컬럼명 [ASC | DESC];
```
  - 컬럼명을 기준을 정렬 가능
2. ORDER BY 컬럼명 ASC : 오름차순 정렬 (ASC는 생략 가능)
3. ORDER BY 컬럼명 DESC : 내림차순 정렬
4. 정렬 기준은 숫자, 문자열, 날짜 등 모든 데이터 타입 컬럼 가능

-----
### 정렬 예제
-----
1. 사원의 번호와 급여를 가져옴. 단, 급여를 기준으로 오름차순 정렬
```sql
SELECT EMP_NO, SALARY
FROM SALARIES
ORDER BY SALARY ASC;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/be954a5d-72aa-43ae-b308-7cfb61b14881">
</div>

2. 사원의 번호와 급여를 가져옴. 단, 급여를 기준으로 내림차순 정렬
```sql
SELECT EMP_NO, SALARY
FROM SALARIES
ORDER BY SALARY DESC;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/b4efe8bc-1236-4344-8bbd-77218c3ecb3b">
</div>

3. 사원의 번호와 이름을 가져옴. 단, 이름을 기준으로 오름차순 정렬
```sql
SELECT EMP_NO, FIRST_NAME
FROM EMPLOYEES
ORDER BY FIRST_NAME ASC;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/0638996a-90db-49e3-9c6f-3e3ae5013ffd">
</div>

4. 사원의 번호와 이름을 가져옴. 단, 이름을 기준으로 내림차순 정렬
```sql
SELECT EMP_NO, FIRST_NAME
FROM EMPLOYEES
ORDER BY FIRST_NAME DESC;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/78e0d189-a32d-4288-bb5d-31d05a139baf">
</div>

5. 사원의 번호와 입사일을 가져옴. 단, 입사일을 기준으로 오름차순 정렬
```sql
SELECT EMP_NO, HIRE_DATE
FROM EMPLOYEES
ORDER BY HIRE_DATE ASC;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/abf87ff2-c0fb-42a4-9d23-71b40c773223">>
</div>

6. 사원의 번호와 입사일을 가져옴. 단, 입사일을 기준으로 내림차순 정렬
```sql
SELECT EMP_NO, HIRE_DATE
FROM EMPLOYEES
ORDER BY HIRE_DATE DESC;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/ead9cb0a-9573-492d-9f36-e09b9180d42d">
</div>
