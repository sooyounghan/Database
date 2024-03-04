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
1. + : 더하기
2. - : 빼기
3. * : 곱하기
4. / : 나누기

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
  - null 값 처리 : NVL2("값", "지정값1", "지정값2") - NVL 함수는 값이 NULL이 아니라면 지정값1을 출력, NULL이면 원래의 값 출력

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
