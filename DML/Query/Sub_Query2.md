-----
### 결과가 하나 이상인 서브 쿼리
-----
1. 서브쿼리를 통해 가져온 결과가 하나 이상인 경우, 결과를 모두 만족하거나 결과 중 하나만 만족해야 하는 경우 존재
2. IN : 서브쿼리 결과 중 하나라도 일치하면 조건은 참

3. EXIST
   - IN과 비슷하지만, 후행 조건절로 값의 리스트가 아닌 서브 쿼리만 가능
   - 서브 쿼리 내 조인 조건이 있어야함
```sql
SEELCT DEPTNO, DNAME
FROM DEPT D
WHERE EXISTS (SELECT *
              FROM EMP E
              WHERE D.DEPTNO = E.DEPTNO
              AND SAL > 3000)
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
WHERE DEPTNO IN (SELECT DEPTNO FROM EMP WHERE SAL >= 3000)
```

2. 직무가 CLERK인 사원과 동일한 부서에 근무하고 있는 사원들의 사원번호, 이름, 입사일을 가져온다.
```sql
SELECT EMPNO, ENAME, HIREDATE
FROM EMP
WEHRE DEPTNO IN (SELECT DEPTNO FROM EMP WHERE JOB = 'CLERK');
```

3. KING을 직속 상관으로 가지고 있는 사람들이 근무하고 있는 근무 부서명, 지역을 가지고 온다.
```sql
SELECT DNAME, LOC
FROM DEPT
WEHRE DEPTNO IN (SELECT DEPTNO
                 FROM EMP
                 WHERE MGR = (SELECT EMPNO
                              FROM EMP
                              WHERE ENAME = 'KING');

