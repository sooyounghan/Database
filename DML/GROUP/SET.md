-----
### 집합 연산자
-----
1. 데이터 집합을 대상으로 연산을 수행하는 연산자
2. 데이터의 집합의 수는 한 개 이상 사용 가능
3. 여러 개의 SELECT문을 연결해 또 다른 하나의 쿼리를 만들 수 있음
4. 즉, 두 SELECT문을 통해 얻어온 결과에 대해 집합 연산을 할 수 있는 명령어
5. 두 SELECT 문을 통해 가져온 컬럼의 형태가 완전히 일치해야 함

-----
### UNION : 합집합
-----
1. 두 개의 데이터 집합이 있으면, 각 집합 원소(SELECT 결과)를 모두 포함한 결과 반환 (중복된 데이터는 한 번만 조회)
 - 10번 부서에 근무하고 있는 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 직무가 CLERK인 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 위 두 가져온 데이터를 합치되, 중복된 행이 있다면 1번만 출력한다.

```sql
SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE DEPTNO = 10

UNION

SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE JOB = 'CLERK';
```

2. UNION ALL : UNION과 비슷하지만, 중복된 항목도 모두 조회
 - 10번 부서에 근무하고 있는 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 직무가 CLERK인 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 위 두 가져온 데이터를 합치되, 중복된 행이 있다면, 중복해서 출력한다.

```sql
SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE DEPTNO = 10

UNION ALL

SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE JOB = 'CLERK';
```

-----
### INTERSECT : 교집합
-----
: 데이터 집합에서 공통된 항목만 추출
 - 10번 부서에 근무하고 있는 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 직무가 CLERK인 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 위 두 가져온 데이터 중 공통되는 데이터만 조회한다.

```sql
SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE DEPTNO = 10

INTERSECT

SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE JOB = 'CLERK';
```

-----
### MINUS : 차집합
-----
: 한 데이터 집합을 기준으로 다른 데이터 집합과 공통된 항목을 제외한 결과만 추출
 - 10번 부서에 근무하고 있는 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 직무가 CLERK인 사원의 사원번호, 이름, 직무, 근무부서 번호를 가져온다.
 - 위 두 가져온 데이터 중 19번 부서에만 존재하는 데이터를 출력한다.

```sql
SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE DEPTNO = 10

MINUS

SELECT EMPNO, ENAME, DEPTNO
FROM EMP
WHERE JOB = 'CLERK';
```

-----
### 집합 연산자의 제한사항
-----
1. 집합 연산자로 연결되는 각 SELECT문의 SELECT 리스트의 개수와 데이터 타입은 일치
2. 집합 연산자로 SELECT 문을 연결할 때, ORDER BY 절은 맨 마지막 문장에서만 사용 가능
3. BLOB, CLOB, BFILE 타입의 컬럼에 대해서는 집합 연산자 사용 불가
4. UNION, INTERSECT, MINUS 연산자는 LONG형 컬럼에 사용할 수 없음
