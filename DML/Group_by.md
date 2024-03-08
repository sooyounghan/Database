-----
### Group by
-----
1. 그룹 함수를 사용할 경우 SELECT ~ FROM ~ WHERE 절까지 모두 수행하여 가져온 결과를 하나의 그룹으로 묶어 총합, 평균 등을 구할 수 있음
2. SELECT문을 수행하여 가져온 하나의 결과를 여러 그룹으로 나눠 그룹 각각의 총합과 평균 등을 구할 수 있음
```sql
SELECT COL_NAME
FROM TABLE_NAME
WHERE 조건절
GROUP BY 그룹 기준
ORDER BY 정렬 기준 정렬 방법
```
3. GROUP BY를 사용할 때, 그룹이 되는 조건을 같이 사용하면 더 명확하게 볼 수 있음

-----
### 쿼리문 실행순서
-----
1. FROM
2. WHERE
3. ORDER BY
4. SELECT
5. DISTINCT
6. GROUP BY
[7. LIMIT]

------
### 예제
------
1. 각 부서별 사원들의 급여 평균을 구한다.
```sql
SELECT DEPTNO, AVG(SAL)
FROM EMP
GROUP BY DEPTNO;
```

2. 각 직무별 사원들의 급여 총합을 구한다.
```sql
SELECT JOB, SUM(SAL)
FROM EMP
GROUP BY JOB;
```

3. 1500이상 급여를 받는 사원들의 부서별 급여 평균을 구한다.
```sql
SELECT DEPTNO, AVG(SAL)
FROM EMP
WHERE SAL >= 1500
GROUP BY DEPTNO;
```

-----
### Having
-----
1. GROUP BY로 묶인 각 그룹들 중 실제 가져올 그룹을 선택할 조건을 작성
2. HAVING 절은 GROUP BY 절의 조건이 됨
   
3. 부서별 평균 급여가 2000이상이면, 부서의 급여 평균을 가져온다.
```sql
SELECT DEPTNO, AVG(SAL)
FROM EMP
GROUP BY DEPTNO
HAVING AVG(SAL) >= 2000;
```

4. 부서별 최대 급여액이 3000이하인 부서의 급여 총합을 가져온다.
```sql
SELECT DEPTNO, SUM(SAL)
FROM EMP
GROUP BY DEPTNO
HAVING  MAX(SAL) <= 3000;
```

5. 부서별 최소 급여액이 1000 이하인 부서에서 직무가 CLERK인 사원들의 급여 총합을 구한다.
```sql
SELECT DEPTNO, SUM(SAL)
FROM EMP
WHERE JOB = 'CLERK'
GROUP BY DEPTNO
HAVING MIN(SAL) <= 1000;
```

6. 각 부서의 급여 최소가 900이상 최대가 10000이하인 부서의 사원들 중 1500 이상의 급여를 받는 사원들의 평균 급여액을 가져온다.
```sql
SELECT DEPTNO, AVG(SAL)
FROM EMP
WHERE SAL >= 1500
GROUP BY DEPTNO
HAVING MIN(SAL) >= 900 AND MAX(SAL) <= 10000;
```
