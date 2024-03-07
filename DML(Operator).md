-----
### DML 연산자 - 산술 연산자
-----
1. (+) : 더하기
2. (-) : 빼기
3. (*) : 곱하기
4. (/) : 나누기
5. (% or mod) : 나머지 연산자

6. 사원들의 급여와 급여액에서 1000을 더한값, 200을 뺀 값, 2를 곱한 값, 2로 나눈 값을 가져옴
```sql
SELECT SAL, (SAL + 1000), (SAL - 200), (SAL * 2), (SAL / 2)
FROM EMP;
```

7. 각 사원의 급여액, 커미션, 급여 + 커미션 액수를 가져옴
```sql
SELECT SAL, COMM, (SAL + COMM)
FROM EMP;
```
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/3552a7a8-cf8c-490a-8469-41542a14640c">
</div>   

  - null 값 : 의미가 없는 값, 저장할 값이 없는 것 (Oracle : 무한대를 의미)

  - null 값 처리 : NVL("값", "지정값") - NVL 함수는 값이 NULL인 경우 지정값을 출력, 아니라면 원래의 값 출력
  - null 값 처리 : NVL2("값", "지정값1", "지정값2") - NVL 함수는 값이 NULL이 아니라면 지정값1을 출력, NULL이면 지정값2 출력

```sql
SELECT SAL, NVL(COMM, 0), SAL + NVL(COMM, 0)
FROM EMP;
```

-----
### DML 연산자 - CONCAT 연산자
-----
1. 문자열을 합치는 연산자 : '문자열' || 컬럼 || "문자열" || 컬럼
2. 사원들의 이름과 직무 양식 : OOO 사원의 담당 직무는 XXX입니다.
```sql
SELECT ENAME || ' 사원의 담당 직무는 ' || JOB || ' 입니다.'
FORM EMP;
```

-----
### DML 연산자 - DISTINCT 키워드
-----
1. SELECT 문을 통해 가져온 모든 Row 중에서 중복된 Row를 제거하는 키워드
```sql
SELECT DISTINCT COLUMN_NAME
FROM TABLE_NAME;
```
2. 사원들이 근무하고 있는 근무 부서의 번호를 가져온다.
```sql
SELECT DISTINCT DEPTNO
FROM EMP;
```

-----
### 조건절과 비교 연산자
-----
1. 테이블 내 모든 Row에 대해 적용
2. 어떤 조건에 맞는 Row에 대해서만 작업을 하고 싶을 때 조건절 이용
3. 형태
```sql
SELECT column_name
FROM table_name
WHERE 조건절;
```
4. SELECT ~ FROM 까지 모든 ROW를 얻어온 뒤, 각 행을 조건절과 비교해서 참 값은 남겨두고, 거짓은 제거
5. 비교 연산자

       - < : lesser than
       - > : grater than
       - <= : lesser equal
       - >= : grater equal
       - = : equal (==가 아님!)
       - !=, ^=, <> : not equal
       - 비교 연산자의 결과는 1(true) 또는 0(false)

7. 근무 부서가 10번인 사원들의 사원번호, 이름, 근무 부서를 가져온다.
```sql
SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE DEPTNO = 10;
```

7. 근무 부서 번호가 10번이 아닌 사원들의 사원번호, 이름, 근무 부서 번호를 가져온다.
```sql
SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE DEPTNO != 10;
(= WHERE DEPTO <>(^=) 10;
```

8. 급여가 1500이상인 사원들의 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE SAL >= 1500;
```

9. 이름이 SCOTT 사원의 사원번호, 이름, 직무, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, JOB, SAL
FROM EMP
WHERE ENAME = 'SOCTT';
```
   - 문자열을 가져올 때는 반드시 ' ' 또는 " " (중요)
   - 🎈 오라클에서는 맞춤법, 대소문자까지 완전히 일치해야만 조회!
     
10. 직무가 SALEMAN인 사원의 사원번호, 이름, 직무를 가져온다
```sql
SELECT EMPNO, ENAME, JOB
FROM EMP
WHERE JOB = 'SALEMAN';
```

11. 직무가 CLERK이 아닌 사원의 사원 번호, 이름, 직무를 가져온다.
```sql
SELECT EMPNO, ENAME, JOB
FROM EMP
WHERE JOB <> 'CLERK';
```

12. 1982년 1월 1일 이후에 입사한 사원의 사원번호, 이름, 입사일을 가져온다.
```sql
SELECT EMPNO, ENAME, HIREDATE
FROM EMP
WHERE HIREDATE >= '1982/01/01';
```
   - 년도는 '년/월/일'로 표기

-----
### 논리 연산자
-----
1. 여러 조건식을 묶어 하나의 조건식으로 처리 가능
2. 종류
  - and : 좌우 조건식이 모두 참이면 참
  - or : 좌우 조건식이 모두 거짓일 경우 거짓
  - not : 조건식 결과 부정
  - between ~ and ~ : 범위 조건
  - in : 항목 조건

3. 10번 부서에서 근무하고 있는 직무가 MANAGER인 사원의 사원번호, 이름, 근무부서, 직무를 가져온다.
```sql
SELECT EMPNO, ENAME, EMPNO, JOB
FROM EMP
WHERE (DEPTNO = 10) AND (JOB = 'MANAGER');
```

4. 입사년도가 1981년인 사원중에 급여가 1500 이상인 사원의 사원번호, 이름, 급여, 입사일을 가져온다.
```sql
SELECT EMPNO, ENAME, SAL, HIREDATE
FROM EMP 
WHERE (SAL >= 1500) AND (HIREDATE BETWEEN '1981/01/01' AND '1981/12/31');
(= WHERE (SAL >= 1500) AND (HIREDATE >= '1981/01/01' AND 'HIREDATE <= 1981/12/31');
```

5. 20번 부서에 근무하고 있는 사원 중 급여가 1500 이상인 사원의 사원번호, 이름, 부서번호, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, DEPNO, SAL
FROM EMP
WHERE (DEPNO = 20) AND (SAL >= 1500);
```

6. 직속상관의 사원 번호가 7698번인 사원 중 직무가 CLERK인 사원의 사원번호, 이름, 직속상관번호, 직무를 가져온다.
```sql
SELECT EMPNO, ENAME, MGR, JOB
FROM EMP
WHERE (MGR = 7698) AND (JOB = 'CLERK');
```

7. 급여가 2000보다 크거나 1000보다 작은 사원번호, 이름, 급여를 가져온다.
```sql
SELECT EMPNO, ENAME, SAL
FROM EMP
WHERE (SAL < 1000) AND (SAL > 2000);
(= WHERE NOT((SAL >= 1000) AND (SAL <= 2000));)
(= WHERE NOT(SAL BETWEEN 1000 AND 2000);) 
```

8. 부서 번호가 20이거나 30인 사원의 사원 번호, 이름, 부서 번호를 가져온다.
```sql
SELECT EMPNO, ENAME, DEPNO
FROM EMP
WHERE EMPNO = 20 OR EMPNO = 30;
```

9. 직무가 CLERK, SALESMAN, ANALYST인 사원의 사원번호, 이름, 직무를 가져온다.
```sql
SELECT EMPNO, ENAME, JOB
FROM EMP
WHERE JOB IN ('CLERK', 'SALESMAN', 'ANALYST');
(= WHERE (JOB = 'CLERK') OR (JOB = 'SALESMAN') OR (JOB = 'ANALYST');)
```

10. 사원 번호가 7499, 7566, 7839가 아닌 사원들의 사원번호, 이름을 가져온다.
```sql
SELECT EMPNO, ENAME
FROM EMP
WHERE NOT(EMPNO IN(7499, 7566, 7839));
(= WHERE EMPNO <> 7499 AND EMPNO <> 7566 AND EMPNO <> 7839);)
(= WHERE NOT (EMPNO = 7499 OR EMPNO = 7566 OR EMPNO = 7839);)
```
  - IN 연산자 수행 -> NOT으로 부정

-----
### 조건식 (Condition)
-----
1. ANY, SOME : '아무것'이나 '하나만'을 뜻하므로, 주어진 값 중 하나라도 존재하는 값 (= OR 연산자와 동일)
```sql
SELECT EMPNO, SAL
FROM EMP
WHERE SAL = ANY|SOME(2000, 3000, 4000)
OREDER BY EMPNO;
```

2. ALL : 모든 조건을 동시에 만족 (= AND 연산자와 동일)
 ```sql
SELECT EMPNO, SAL
FROM EMP
WHERE SAL = ALL(2000, 3000, 4000)
OREDER BY EMPNO;
```
   
