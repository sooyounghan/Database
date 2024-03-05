------
### 정렬
------
1. SELECT 문을 통해 얻은 결과를 특정 컬럼 기준으로 오름차순 / 내림차순 정렬 가능
2. 숫자, 문자열, 날짜 등 모든 타입에 대해 데이터 정렬 가능
3. 형식
```sql
SELECT COL_NAME
FROM TABLE_NAME
WHERE 조건절
ORDER BY 정렬조건 [ASC|DESC]
```

  - ASC : 오름차순 (생략 가능)
  - DESC : 내림차순

4. 테이블에 대해 모든 ROW에 대해 SELECT에 의해 컬럼을 추출 - WHERE절에 따라 조건에 따라 컬럼 추출 - ORDER BY 절에 의해 정렬

5. 사원의 사원번호, 이름, 급여를 가져온다. 급여를 기준으로 오름차순 정렬을 한다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
ORDER BY SAL ASC;
```

6. 사원의 사원번호, 이름, 급여를 가져온다. 사원 번호를 기준으로 내림차순으로 정렬한다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
ORDER BY EMPNO DESC;
```

7. 사원의 사원번호, 이름을 가져온다. 사원의 이름을 기준으로 오름차순 정렬을 한다. (문자열 타입 컬럼도 정렬 가능)
```sql
SELECT EMPNO, ENAME
FROM EMP
ORDER BY ENAME ASC;
```

8. 사원의 입사번호, 이름, 입사일을 가져온다. 입사일을 기준으로 내림차순 정렬을 한다. (날짜 타입 컬럼도 정렬 가능)
```sql
SELECT EMPNO, ENAME, HIREDATE
FROM EMP
ORDER BY HIREDATE DESC;
```

9. 직무가 SALESMAN인 사원의 사원이름, 사원번호, 급여을 가져온다. 급여를 기준으로 오름차순 정렬을 한다.
```sql
SELECT ENAME, EMPNO, SAL
FROM EMP
WHERE JOB = 'SALESMAN'
ORDER BY SAL ASC;
```

10. 1981년 입사한 사원들의 사원번호, 사원 이름, 입사일을 가져온다. 사원 번호를 기준으로 내림차순 정렬을 한다.
```sql
SELECT EMPNO, ENAME, HIREDATE
FROM EMP
WHERE HIREDATE BETWEEN '1981/01/01' AND '1981/12/31'
ORDER BY EMPNO DESC;
```

11. 사원의 이름, 급여, 커미션을 가져온다. 커미션을 기준으로 오름차순 정렬을 한다.
    - COMM 컬럼에는 NULL 값 존재 : NULL은 무한대의 값이므로 어떠한 값보다 크게 됨
    - COMM 컬럼에는 NULL 값이 존재하므로, NULL 값을 제외해도 가능
```sql
SELECT ENAME, SAL, COMM
FROM EMP
[WHERE COMM IS NOT NULL]
ORDER BY COMM ASC;
```

11. 사원의 이름, 급여, 커미션을 가져온다. 커미션을 기준으로 내림차순 정렬을 한다.
```sql
SELECT ENAME, SAL, COMM
FROM EMP
[WHERE COMM IS NOT NULL]
ORDER BY COMM DESC;
```

12. 사원의 이름, 사원번호, 급여를 가져온다. 급여를 기준으로 내림차순 정렬, 이름을 기준으로 오름차순 정렬한다.
```sql
SELECT ENAME, EMPNO, SAL
FROM EMP
ORDER BY SAL DESC, ENAME ASC;
```
 - ORDER BY 정렬기준1 정렬방법, 정렬기준2 정렬방법, 정렬기준3 정렬방법 ... 정렬기준n 정렬방법 : n차 정렬 가능
 - 의미 : 정렬기준1에 의해 정렬을 하고, 이에 대해 정렬기준2에 대해 정렬을 실시한다.   
   (즉, 이럴 경우 정렬기준1에서 동일한 값이 나오면, 정렬기준2에 대해 또 다른 정렬 가능)
