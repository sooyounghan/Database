-----
### 결과가 하나 이상인 서브 쿼리
-----
1. 서브쿼리를 통해 가져온 결과가 하나 이상인 경우, 결과를 모두 만족하거나 결과 중 하나만 만족해야 하는 경우 존재
2. IN : 서브쿼리 결과 중 하나라도 일치하면 조건은 참

3. EXIST
   - IN과 비슷하지만, 후행 조건절로 값의 리스트가 아닌 서브 쿼리만 가능
   - 서브 쿼리 내 조인 조건이 있어야함
```sql
SELECT DEPTNO, DNAME
FROM DEPT D
WHERE EXISTS (SELECT *
              FROM EMP E, DEPT D
              WHERE D.DEPTNO = E.DEPTNO
              AND E.SAL > 3000)
```

5. ANY, SOME : 서브쿼리의 결과와 하나 이상 일치하면 조건은 참
6. ALL : 서브쿼리의 결과와 모두 일치해야 조건은 참

-----
### 예제
-----
1. 3000 이상의 급여를 받는 사원들과 같은 부서에 근무하고 있는 사원의 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE DEPNTO IN (SELECT DEPTNO
                  FROM EMP
                  WHERE SAL >= 3000);
```
   - DEPTNO = (SELECT..)로 설정하면 서브쿼리는 1개 이상의 값을 반환하므로 IN 설정

2. 직무가 CLERK인 사원과 동일한 부서에 근무하고 있는 사원들의 사원번호, 이름, 입사일을 가져온다.
```sql
SELECT EMPNO, ENAME, HIREDATE
FROM EMP
WHERE DEPTNO IN (SELECT DEPTNO
                 FROM EMP
                 WHERE JOB = 'CLERK');
```

3. KING을 직속 상관으로 가지고 있는 사람들이 근무하고 있는 근무 부서명, 지역을 가지고 온다.
```sql
SELECT DNAME, LOC
FROM DEPT
WHERE DEPTNO IN (SELECT DEPTNO
                 FROM EMP
                 WHERE MGR = (SELECT EMPNO
                              FROM EMP
                              WHERE ENAME = 'KING'); 
```

4. CLERK들의 직속상관의 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE EMPNO IN (SELECT MGR
                 FROM EMP
                 WHERE JOB = 'CLERK');
```

5. 각 부서별 급여평균보다 더 많이 받는 사원의 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE SAL > ALL (SELECT AVG(SAL)
                 FROM EMP
                 GROUP BY DEPTNO);
```
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE SAL > (SELECT MAX(AVG(SAL))
             FROM EMP
             GROUP BY DEPTNO);
```

6. 각 부서별 급여 최저치보다 더 많이 받는 사원들의 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE SAL > ALL(SELECT MIN(SAL)
                FROM EMP
                GROUP BY DEPTNO);
```
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE SAL >  SELECT MAX(MIN(SAL))
             FROM EMP
             GROUP BY DEPTNO);
```

7. SALESMAN 보다 급여를 적게 받는 사원들의 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SLA
FROM EMP
WHERE SAL < ALL (SELECT SAL
                 FROM EMP
                 WHERE JOB = 'SALESMAN');
```
```sql
SELECT EMPNO, ENAME, SLA
FROM EMP
WHERE SAL < (SELECT MIN(SAL)
             FROM EMP
             WHERE JOB = 'SALESMAN');
```

8. 각 부서별 최저 급여 액수보다 많이 받는 사원들의 사원번호, 이름, 급여를 가져온다.
```sql 
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE SAL > ANY (SELECT MIN(SAL)
                FROM EMP
                GROUP BY DEPTNO);
```

9. DALLAS에 근무하고 있는 사원들 중 가장 나중에 입사한 사원의 날짜보다 더 먼저 입사한 사원들의 사원번호, 이름, 입사일을 가져온다.
```sql
SELECT EMPNO, ENAME, HIREDATE
FROM EMP 
WHERE HIREDATE < ALL(SELECT HIREDATE
                     FROM EMP
                     WHERE DEPTNO = (SELECT DEPTNO
                                     FROM DEPT
                                     WHERE LOC = 'DALLAS'));
```

