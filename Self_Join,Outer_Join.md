-----
### Self Join
-----
1. 같은 테이블에 두 번 이상 조인하는 것

2. SMITH 사원의 사원번호, 이름, 직속상관 이름을 가져온다.
   - E1 : SMITH의 사원 정보
   - E2 : SMITH 사원의 직속 상관 정보
```sql
SELECT E1.EMPNO, E1.ENAME, E1, E2.ENAME
FROM EMP E1, EMP E2 
WHERE (E1.MGR = E2.EMPNO) AND (E1.ENAME = 'SMITH');
```

3. FROD 사원 밑에서 일하는 사원들의 사원번호, 이름, 직무를 가져온다.
  - E1 : FROD 사원의 부하직원 정보
  - E2 : FROD 사원의 사원 정보
```sql
SELECT E1.EMPNO, E1.ENAME, E1.JOB
FROM EMP E1, EMP E2
WHERE (E1.MGR = E2.EMPNO) AND (E2.ENAME = 'FORD');
```

