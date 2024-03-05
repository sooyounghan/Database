-----
### SELECT문
-----
1. 데이터베이스에 저장된 데이터를 가져오는 명령문
2. 상황이나 조건에 맞는 데이터를 추출 가능
3. 기본 구조

```sql
SELECT {* | 컬럼명 1, .. 컬럼명 N}
FROM 테이블명
[WHERE 조건]
[GROUP BY 그룹기준]
[HAVING 그룹조건]
[ORDER BY 정렬 기준 정렬 방법];
```

<div align = "center">
<img src="https://github.com/sooyounghan/Data-Structure/assets/34672301/7ce05962-a93d-4379-a93d-fb60058f5bf5">
</div>

-----
### SELECT - 모든 Column의 데이터 가져오기
-----
* 항상 키워드 - FROM절 - 나머지 적기
  
```sql
SELECT *
FROM TABLE_NAME;
```
1. < DEPT > TABLE의 모든 정보를 가져옴
```sql
SELECT *
FROM DEPT;
```

2. < EMP > TABLE의 모든 정보를 가져옴
```sql
SELECT *
FROM EMP;
```

-----
### SELECT - 특정 Column의 데이터 가져오기
-----
```sql
SELECT COL1, COL2(COL1과 COL2는 TABLE에 있는 COLUMN)
FROM TABLE_NAME;
```

1. < EMP > TABLE에서 사원의 이름과 사원 번호를 가져옴
```sql
SELECT ENAME, EMPNO
FROM EMP;
```

2. < EMP > TABLE에서 사원의 이름과 사원 번호, 직무, 급여를 가져옴
```sql
SELECT ENAME, EMPNO, JOB, SAL
FROM EMP;
```

3. < DEPT > TABLE에서 부서 번호와 부서 이름을 가져옴
```sql
SELECT DEPNO, DANME
FROM DEPT;
```

-----
### DML 연산자 - 산술 연산자
-----
1. (+) : 더하기
2. (-) : 빼기
3. (*) : 곱하기
4. (/) : 나누기

5. 사원들의 급여와 급여액에서 1000을 더한값, 200을 뺀 값, 2를 곱한 값, 2로 나눈 값을 가져옴
```sql
SELECT SAL, (SAL + 1000), (SAL - 200), (SAL * 2), (SAL / 2)
FROM EMP;
```

6. 각 사원의 급여액, 커미션, 급여 + 커미션 액수를 가져옴
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

6. 근무 부서가 10번인 사원들의 사원번호, 이름, 근무 부서를 가져온다.
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
