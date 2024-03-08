-----
### Self Join
-----
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/e4503ba0-e62d-4bab-9e38-2278ffc4b744">
</div>
   
1. 같은 테이블에 두 번 이상 조인하는 것

2. SMITH 사원의 사원번호, 이름, 직속상관 이름을 가져온다.
   - E1 : SMITH의 사원 정보
   - E2 : SMITH 사원의 직속 상관 정보
```sql
SELECT E1.EMPNO, E1.ENAME, E2.ENAME
FROM EMP E1, EMP E2
WHERE (E1.MGR = E2.EMPNO) AND (E1.ENAME = 'SMITH');
```

3. FROD 사원 밑에서 일하는 사원들의 사원번호, 이름, 직무를 가져온다.
  - E1 : FROD 사원의 사원 정보
  - E2 : FROD 사원의 부하직원 정보
```sql
SELECT E2.EMPNO, E2.ENAME, E2.JOB
FROM EMP E1, EMP E2
WHERE (E1.EMPNO = E2.MGR) AND (E1.ENAME = 'FROD'); 
```

4. SMITH 사원의 직속상관과 동일한 직무를 가지고 있는 사원들의 사원번호, 이름, 직무를 가져온다.
   - E1 : SMITH 사원의 사원 정보
   - E2 : SMITH 사원의 직속 상관 정보
   - E3 : SMITH 사원의 직속 상관과 동일한 직무를 가지고 있는 사원의 정보
      
```sql
SELECT E3.EMPNO, E3.ENAME, E3.JOB
FROM EMP E1, EMP E2, EMP E3
WHERE (E1.MGR = E2.EMPNO) AND (E1.ENAME = 'SMITH') AND (E2.JOB = E3.JOB);
```

-----
### Outer Join
-----
1. 조인 조건에 해당하지 않기 때문에 결과에 포함되지 않는 행(Row)까지 가져오는 조인 : 정보가 없는 부분에 대해 OUTER JOIN

2. 각 사원의 이름, 사원번호, 직장 상사의 이름을 가져온다. 단, 직속 상관이 없는 사원도 가져온다.
   - E1 : 각 사원의 정보
   - E2 : 직장 상사의 정보 (직장 상사 중 KING은 직장 상사 번호가 없으므로 NULL 이므로 이 정보 포함해야함) : 없는 부분에 OUTER JOIN 표시
```sql
SELECT E1.ENAME, E1.EMPNO, E2.ENAME
FROM EMP E1, EMP E2
WHERE E1.MGR = E2.EMPNO(+);
```

3. 모든 부서의 소속 사원의 근무부서명, 사원번호, 사원이름, 급여를 가져온다.
   - D : 부서의 정보
   - E : 사원의 정보 (사원의 근무부서 번호 중 40번은 없음) : 없는 부분에 OUTER JOIN 표시
```sql
SELECT D.DNAME, E.EMPNO, E.ENAME, E.SAL
FROM DEPT D, EMP E
WHERE D.DEPTNO = E.DEPTNO(+);
```
