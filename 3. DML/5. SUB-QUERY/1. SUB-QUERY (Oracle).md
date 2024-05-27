-----
### 서브쿼리 (Sub Query)
-----
1. Query문 안에 들어가는 Query문
2. Query문 작성 시 사용되는 값을 다른 Query문을 통해 구해야 할 경우 사용

3. SCOTT 사원이 근무하고 있는 부서의 이름을 가져온다.
```sql
SELECT DNAME
FROM DEPT
WHERE DEPTNO = (SELECT DEPTNO
                FROM EMP
                WHERE ENAME = 'SCOTT');
```
```sql
SELECT D.DNAME
FROM EMP E, DEPT D
WHERE (D.DEPTNO = E.DEPTNO) AND (E.NAME = 'SCOTT')
```

4. SMITH와 같은 부서에 근무하고 있는 사원들의 사원번호, 이름, 급여액, 부서이름을 가져온다.
```sql
SELECT E.EMPNO, E.ENAME, E.SAL, D.DNAME 
FROM EMP E, DEPT D
WHERE (E.DEPTNO = D.DEPTNO) AND (E.DEPTNO = (SELECT DEPTNO
                                                    FROM EMP
                                                    WHERE ENAME = 'SMITH'));
```

5. MARTIN과 같은 직무를 가지고 있는 사원들의 사원번호, 이름, 직무를 가져온다.
```sql
SELECT EMPNO, ENAME, JOB
FROM EMP
WHERE JOB = (SELECT JOB
             FROM EMP
             WHERE ENAME = 'MARTIN');
```

6. ALLEN과 같은 직속상관을 가진 사원들의 사원번호, 이름, 직속상관 이름을 가져온다.
   - EMP 미지정 : ALLEN의 정보
   - E1 : ALLEN과 같은 직속상관을 가진 사원들에 대한 정보
   - E2 : ALLEN의 직속 상관에 대한 정보
     
```sql
SELECT E1.EMPNO, E1.ENAME, E2.ENAME
FROM EMP E1, EMP E2
WHERE (E1.MGR = E2.EMPNO) AND (E1.MGR = (SELECT MGR
                                         FROM EMP
                                         WHERE ENAME = 'ALLEN'));
```

7. WARD와 같은 부서에 근무하고 있는 사원들의 사원번호, 이름, 부서번호를 가져온다.
```sql
SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE DEPTNO = (SELECT DEPTNO
                FROM EMP
                WHERE NAME = 'WARD'));
```

8. SALESMAN의 평균 급여보다 많이 받는 사원들의 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE SAL > (SELECT AVG(SAL)
              FROM EMP
              WHERE JOB = 'SALESMAN);
```

9. DALLAS 지역에 근무하는 사원들의 평균 급여를 가져온다.
```sql
SELECT AVG(SAL)
FROM EMP
WHERE (DPETNO = (SELECT DEPTNO
                 FROM DEPT
                 WHERE LOC = 'DALLAS'));
```

10. SALES 부서에 근무하는 사원들의 사원번호, 이름, 근무지역을 가져온다.
```sql
SELECT E.EMPNO, E.ENAME, D.LOC
FROM EMP E, DEPT D
WHERE (E.DEPTNO = D.DEPTNO) AND E.DEPTNO = (SELECT DEPTNO
                                            FROM DEPT
                                            WHERE DNAME = 'SALES');
```

11. CHICAGO 지역에 근무하는 사원들 중 BLAKE이 직속상관인 사원들의 사원번호, 이름, 직무를 가져온다.
    - E1 : BLAKE이 직속상관인 사람들의 사원 정보
    - E2 : BLAKE 사원에 대한 정보
      
```sql
SELECT EMPNO, ENAME, JOB
FROM EMP
WHERE MGR = (SELECT EMPNO 
             FROM EMP                        
             WHERE ENAME = 'BLAKE')
AND DEPTNO = (SELECT DEPTNO
              FROM DEPT
              WHERE LOC = 'CHICAGO');
````
