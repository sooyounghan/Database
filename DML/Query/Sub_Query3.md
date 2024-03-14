-----
### 스칼라 서브쿼리 (Scalar SubQuery)
-----
1. Sub-Query가 SELECT절에 포함된 경우
2. 조건 : 서브쿼리가 단 1개(= 1개의 ROW)
3. 급여가 3000이상인 사번, 사원명, 부서번호, 부서명를 조회한다.
```sql
SELECT E.EMPNO, E.ENAME, E.SAL, E.DEPTNO,
         (SELECT D.DNAME 
          FROM DEPT D 
          WHERE D.DEPTNO = E.DEPTNO) AS "DEPT_NAME"
FROM EMP E
WHERE E.SAL >= 3000;
```

4. 20번 부서의 사번, 사원명, 직무, 급여, 동일한 직무의 평균 급여액과이 차액을 조회한다. (단, 직무는 오름차순, 동일 직무라면 사원번호 오름차순)
```sql
SELECT E1.EMPNO, E1.ENAME, E1.JOB, E1.SAL, 
          E1.SAL - (SELECT TRUNC(AVG(E2.SAL)) 
                      FROM EMP E2 
                      WHERE E1.JOB = E2.JOB) AS "SAL2"  
FROM EMP E1
WHERE E1.DEPTNO = 20
ORDER BY E1.JOB ASC, E1.EMPNO ASC;
```

5. MANAGER가 있는 부서번호, 부서명, 존재여부 출력한다.
```sql
SELECT D.DEPTNO, D.DNAME,
          CASE WHEN D.DEPTNO IN (SELECT E.DEPTNO 
			        FROM EMP E
			        WHERE E.JOB = 'MANAGER')
 	      THEN 'O'
	      ELSE 'X'
          END AS "존재 여부"
FROM DEPT D;
```
-----
### 인라인 뷰 (Inline View)
-----
1. FROM 절 내에 들어가는 SubQuery

	- FROM 절은 본래 Table나 View가 올 수 있으나, 서브 쿼리를 FROM 절에 사용 가능
 	- 결과적으로 View를 분해하면, 하나의 독립적인 SELECT문 이므로 FROM 절에 사용하는 서브 쿼리도 하나의 View로 볼 수 있기 때문에 붙여진 이름

2. EMP 테이블의 사원번호, 사원이름, 입사일에 대해 입사일을 기준으로 내림차순으로 정렬하여 조회한다.
```sql
SELECT A.*
FROM (SELECT EMPNO, ENAME, HIREDATE
         FROM EMP) A
ORDER BY A.HIREDATE DESC;
```


