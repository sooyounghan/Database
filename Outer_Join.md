-----
### Outer Join
-----
1. 조인 조건에 해당하지 않기 때문에 결과에 포함되지 않는 행(Row)까지 가져오는 조인 : 정보가 없는 부분에 대해 OUTER JOIN
2. 즉, 조인 조건에 만족하는 데이터 뿐만 아니라, 어느 한 쪽 테이블에 조인 조건에 명시된 컬럼 갑이 없거나(NULL 이더라도) 해당 로우가 아예 없는 경우라도 추출

3. 각 사원의 이름, 사원번호, 직장 상사의 이름을 가져온다. 단, 직속 상관이 없는 사원도 가져온다.
   - E1 : 각 사원의 정보
   - E2 : 직장 상사의 정보 (직장 상사 중 KING은 직장 상사 번호가 없으므로 NULL 이므로 이 정보 포함해야함) : 없는 부분에 OUTER JOIN 표시
```sql
SELECT E1.ENAME, E1.EMPNO, E2.ENAME
FROM EMP E1, EMP E2
WHERE E1.MGR = E2.EMPNO(+);
```

4. 모든 부서의 소속 사원의 근무부서명, 사원번호, 사원이름, 급여를 가져온다.
   - D : 부서의 정보
   - E : 사원의 정보 (사원의 근무부서 번호 중 40번은 없음) : 없는 부분에 OUTER JOIN 표시
```sql
SELECT D.DNAME, E.EMPNO, E.ENAME, E.SAL
FROM DEPT D, EMP E
WHERE D.DEPTNO = E.DEPTNO(+);
```
