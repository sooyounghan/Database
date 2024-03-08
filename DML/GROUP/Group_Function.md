-----
### 그룹 함수
-----
1. SELECT문을 통해 가져올 결과를 그룹으로 묶고, 그룹 내에서 지정된 컬럼의 총합, 평균 등을 구할 수 있는 함수
   - SUM(expr) : 총 합계 값
   - AVG(expr) : 평균값
   - COUNT(expr, *) : 쿼리 결과의 전체 Row 수 반환
   - MAX(expr) : 최대 값
   - MIN(expr) : 최소 값
   - VAR(expr) : 분산 (주어진 범위의 개별 값과 평균값의 차이인 편차를 구해 제곱해서 평균한 값)
   - STDDEV(expr) : 표준 편차 (분산 값의 제곱근)
2. 집계함수는 NULL값을 포함하지 않음! (NULL은 무한이며, 값이 없는 상태를 의미)
   
3. COUNT 함수
    - (*) : '테이블의 모든 행'에 대해서 확인하므로 이에 대해 집계
    - column : 지정한 열에서 확인하는 것이며, 내부적으로 NULL이 아닌 값은 제외하고 그 개수를 세는 것
    
4. 사원들의 급여 총합을 구한다.
```sql
SELECT SUM(SAL)
FROM EMP;
```

  - 단, SUM() 함수는 결과 값이 함수이므로 다음과 같은 쿼리문 실행 시 오류가 발생
```sql
SELECT EMPNO, SUM(SAL)
FROM EMP;
```
  - EMPNO은 개수가 여러개(사원 전체에 대한 행) 이며, SUM은 1개이므로 개수가 맞지 않아 오류가 발생
    
4. 사원들의 커미션의 합을 가져온다.
```sql
SELECT SUM(COMM)
FROM EMP
```

5. 급여가 1500 이상인 사람들의 급여의 총합
```sql
SELECT SUM(SAL)
FROM EMP
WHERE SAL >= 1500;
```

6. 20번 부서에 근무하고 있는 사원들의 급여 총합을 구한다.
```sql
SELECT SUM(SAL)
FROM EMP
WHERE EMP = 20;
```

7. 직무가 SALESMAN인 사원들의 급여 총합을 구한다.
```sql
SELECT SUM(SAL)
FROM EMP
WHERE JOB = 'SALESMAN';
```

8. 직무가 SALESMAN인 사원들의 이름과 급여 총합을 가져온다. (오류 발생)
```sql
SELECT DNAME, SUM(SAL)
FROM EMP
WHERE JOB = 'SALESMAN';
```

  - DNAME은 개수가 여러개(사원 전체에 대한 행) 이며, SUM은 1개이므로 개수가 맞지 않아 오류가 발생

9. 전 사원의 급여 평균을 구한다.
```sql
SELECT AVG(SAL)
FROM EMP;
```

10. 커미션을 받는 사람들의 커미션 평균을 구한다.
```sql
SELECT AVG(COMM)
FROM EMP
[= WHERE COMM IS NOT NULL];
```

11. 전 사원의 커미션 평균을 구한다.
```sql
SELECT AVG(NVL(COMM, 0))
FROM EMP
```
  - NULL인 값을 제외하고 평균을 구하므로, NULL을 포함시켜줘야하므로 NVL 함수(영구적인 것이 아님 SELECT문에만 한정) 이용

12. 커미션을 받는 사원들의 급여 평균을 구한다.
```sql
SELECT AVG(SAL)
FROM EMP
WHERE COMM IS NOT NULL;
```

12. 30번 부서에 근무하고 있는 사원들의 급여 평균을 구한다.
```sql
SELECT AVG(SAL)
FROM EMP
WHERE DEPT = 30;
```

13. 직무가 SALESTMAN인 사원들의 급여 + 커미션 평균을 구한다.
```sql
SELECT AVG(SAL + NVL(COMM, 0))
FROM EMP
WHERE JOB = 'SALESMAN';
```

14. 사원들의 총 수를 가져온다.
```sql
SELECT COUNT(EMPNO)
FROM EMP;
```
```sql
SELECT COUNT(*)
FROM EMP;
```

15. 커미션을 받는 사원들의 총 수를 가져온다.
```sql
SELECT COUNT(COMM)
FROM EMP;
```

16. 사원들의 급여 최대, 최소값을 가져온다.
```sql
SELECT MAX(SAL), MIN(SAL)
FROM EMP;
```
