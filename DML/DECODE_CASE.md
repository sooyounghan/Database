-----
### DECODE
-----
1. 값에 따라 변환값이 결정되는 구문
2. 형식
```sql
DECODE(COL_NAME, expr1, result1, expr2, result2, ... , default) DECODES
```
3. 각 사원의 부서 이름을 가져온다.
   - 10 : 인사과
   - 20 : 개발부
   - 30 : 경영지원팀
   - 40 : 생산부
```sql
SELECT EMPNO, ENAME,
       DECODE(DEPTNO, 10, '인사과',
                      20, '개발부',
                      30, '경영지원팀',
                      40, '생산부)
FROM EMP;
```

4. 직급에 따라 인상된 급여액을 가져온다.
   - CLERK : 10%
   - SALESMAN : 15%
   - PRESIDENT : 200%
   - MANAGER : 5%
   - ANALYST : 20%

```sql
SELECT EMPNO, ENAME, JOB DECODE(JOB, 'CLERK', SAL * 1.1,
                                      'SALESMAN', SAL * 1.15,
                                      'PRESIDENT', SAL * 2,
                                      'MAMAGER', SAL * 1.05,
                                      'ANALYST', SAL * 1.2)
FROM EMP;
```

-----
### CASE
-----
1. 조건에 따라 반환 값이 결정되는 구문
2. 형식 (콤마 없음 주의)
```sql
CASE WHEN 조건식1 THEN 반환값1
      WHEN 조건식2 THEN 반환값2
      ...
      ELSE 기타값
END
```
3. THEN 이하 출력 값들의 데이터 타입은 반드시 일치시켜야 함
4. THEN 이하 'A', 'B', 'C' 등 문자 타입이면, 모두 일치

5. 급여액 별 등급을 가져온다.
   - 1000미만 : C등급
   - 1000이상 2000미만 : B등급
   - 2000이상 : A등급
```sql
SELECT EMPNO, ENAME, SAL,
    CASE WHEN SAL < 1000 THEN 'C등급'
         WHEN (SAL >= 1000 AND SAL < 2000) THEN 'B등급'
         WHEN SAL >= 2000 THEN 'A등급'
    END
FROM EMP;
```

6. 직원들의 급여를 다음과 같이 인상한다.
   - 1000 이하 : 100 %
   - 1000 초과 2000 미만 : 50 %
   - 2000 이상 : 200 %
```sql
SELECT EMPNO, ENAME,
      CASE WHEN SAL <= 1000 THEN SAL * 2
           WHEN (SAL > 1000) AND (SAL < 2000) THEN SAL * 1.5
           WHEN (SAL >= 2000) THEN SAL * 3
      END
FROM EMP;
```
