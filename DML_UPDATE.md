-----
### UPDATE
-----
1. ROW 내 COLUMN 값을 수정하는 구문
2. 형식
```sql
UPDATE TABLE_NAME
SET COL_NAME1 = VALUE1, COL_NAME2 = VALUE2, ...;
[WHERE 조건문];
```
3. 예제
   - EMP01 테이블을 만들자.
```sql
CREATE TABLE EMP01
AS
SELECT * FROM EMP;
```

  - 사원들의 부서번호를 40으로 변경한다.
```sql
UPDATE EMP01
SET DEPTNO = 40;
```

  - 사원들의 입사일을 오늘 일자로 변경한다. (sysdate = 현재 일자)
```sql
UPDATE EMP01
SET HIREDATE = SYSDATE;
```

  - 사원들의 직무를 개발자로 모두 변경한다.
```sql
UPDATE EMP01
SET JOB = '개발자';
```

-----
### UPDATE : 여러 열을 변경
-----
1. EMP01 테이블은 다음과 같다.
```sql
CRREATE TABLE EMP01
AS
SELECT * FROM EMP;
```

2. 사원들의 부서번호를 40, 입사일을 오늘, 직무를 개발자로 변경한다.
```sql
UPDATE EMP01
SET DEPTNO = 40, HIREDATE = SYSDATE, JOB = '개발자';
```

-----
### UPDATE : 조건절 추가 (SET : 서브쿼리도 사용 가능)
-----
1. 조건절을 추가하면, 원하는 열의 값만 변경이 가능
2. EMP01 테이블은 다음과 같다.
```sql
CREATE TABLE EMP01
AS
SELECT * FROM EMP;
```

3. 10번 부서에 근무하고 있는 사원들을 40번 부서로 변경한다.
```sql
UPDATE EMP01
SET DEPTNO = 40
WHERE DEPTNO = 10;
```

4. SALESMAN들의 입사일을 오늘 일자로, 커미션을 2000으로 수정한다.
```sql
UPDATE EMP01
SET HIREDATE = SYSDATE, COMM = 2000
WHERE JOB = 'SALESMAN';
```

5. 전체 사원의 평균 급여보다 낮은 사원들의 급여를 50% 인상한다.
```sql
UPDATE EMP01
SET SAL = SAL * 1.5
WHERE SAL < (SELECT AVG(SAL) FROM EMP);
```

6. MANAGER 사원의 직무를 DEVELOPER로 변경한다.
```sql
UPDATE EMP01
SET JOB = 'DEVELOPER'
WHERE JOB = 'MANAGER';
```

7. 30번 부서에 근무하고 있는 사원들의 급여를 전체 평균 급여로 설정한다.
```sql
UPDATE EMP01
SET SAL = (SELECT AVG(SAL) FROM EMP)
WHERE DEPTNO = 30;
```

8. BLAKE 밑에서 근무하고 있는 사원들의 부서를 DALLAS 지역 부서 번호로 변경한다.
```sql
UPDATE EMP01
SET DEPTNO = (SELECT DEPNTO FROM DEPT WHERE LOC = 'DALLAS')
WHERE MGR = (SELECT EMPNO FROM EMP WHERE ENAME = 'BLAKE');
```

9. 20번 부서에 근무하고 있는 사원들의 직속상관을 KING으로 하고, 급여를 전체급여의 최고액으로 설정한다.
```sql
UPDATE EMP01
SET MGR = (SELECT EMPNO FROM EMP WHERE ENAME = 'KING), SAL = (SELECT MAX(SAL) FROM EMP)
WHERE DEPTNO = 20;
```

-----
### UPDATE 예제 심화
-----
1. EMP01 테이블을 다음과 같다.
```sql
CREATE TABLE EMP01
AS
SELECT * FROM EMP;
```

2. 직무가 CLERK인 사원들의 직무와 급여액을 20번 부서에 근무하고 있는 사원 중 급여 최고액을 받는 사원의 직무와 급여액으로 변경한다.
```sql
UPDATE EMP01
SET (SAL, JOB) = (SELECT SAL, JOB FROM EMP WHERE SAL = (MAX(SAL) FROM EMP WHERE DEPTNO = 20))
WHERE JOB = 'CLERK';
```

 
