-----
### INSEERT
-----
1. 테이블에 새로운 Row(Tuple, Record)을 추가하는 구문
2. INSERT INTO TABLE_NAME[(COL_NAME1, COL_NAME2 ... )] VALUES (VALUE1, VAULE2, ...)
   - COL_NAME1과 VALUE1, 이처럼 서로가 1:1로 매칭
   - 컬럼명의 순서와 개수, 타입(길이와 크기도 주의)에 맞추어 값을 입력해야 함
     
3. 다음과 같이 EMP01 TABLE과 구조가 존재한다고 하자. (구조만 복사하는 쿼리문)
```sql
CREATE TABLE EMP01
AS
SELECT EMPNO, ENAME, JOB
FROM EMP
WHERE 1=0; (조건이 거짓이므로, 데이터는 가져오지 못해도 속성은 가져옴)
```

```sql
EMPNO NUMBER(4, 0)
ENAME VARCHAR(10)
JOB VARCHAR2(9)
```

4. 다음과 같은 사원 정보를 입력한다.
   - 사원번호가 1111, 이름이 홍길동, 직무가 인사인 사원의 정보를 입력
```sql
INSERT INTO EMP01(EMPNO, ENAME, JOB) VALUES(1111, '홍길동', '인사');
```

  - 사원번호가 2222, 이름이 김길동, 직무가 개발인 사원의 정보를 입력
```sql
INSERT INTO EMP01(EMPNO, ENAME, JOB) VALUES (2222, '김길동', '개발');
```

  - 사원번호가 3333, 이름이 최길동, 직무가 인사인 사원의 정보를 입력
```sql
INSERT INTO EMP01(EMPNO, ENMAE, JOB) VALUES(3333, '최길동', '인사');
```

  - 사원번호가 4444, 이름이 박길동, 직무가 생산인 사원의 정보를 입력
```sql
INSERT INTO EMP01(EMPNO, ENAME, JOB) VALUES(4444, '박길동', '생산');
```

5. 1개의 테이블에 다중 Record를 입력할 수 있음
```sql
INSERT INTO TABLE_NAME[(COL_NAME1, COL_NAME2, ...)]
VALUES(VALUE1, .... )
VALUES(VALUE1, .... )
...;
```

-----
### INSEERT : COLUMN 목록을 생략하는 경우
-----
1. INSERT INTO TABLE_NAME VALUES (VALUE1, VAULE2, ...)
   : 생략되어도 모든 VALUE 항목대로 각 테이블 항목에 넣을 경우에 가능하며, 1:1매칭

2. 다음과 같은 사원 정보를 입력
   - 사원번호가 5555, 이름이 황길동, 직무가 개발인 사원의 정보를 입력
```sql
INSERT INTO EMP01 VALUES(5555, '황길동', '개발');
```

-----
### INSEERT : COLUMN 목록에 모든 목록이 있지 않은 경우
-----
1. 다음과 같은 사원 정보를 입력
   - 사원번호가 6666, 이름이 이길동인 사원의 정보를 입력
```sql
INSERT INTO EMP01(EMPNO, ENAME) VALUES(6666, '이길동');
```
  : JOB Attribute에는 NULL 값이 입력   

2. 지정해주지 않으면 NULL 값이 삽입되며, 해당 속성 값이 삽입되지 않으면 NULL값 부여

3. 명시적으로 NULL값 지정 가능
   - 사원번호가 7777, 이름이 고길동인 사원의 정보를 입력
```sql
INSERT INTO EMP01(EMPNO, ENAME, JOB) VALUES(7777, '고길동', NULL);
```

-----
### INSEERT : Sub-Query로 데이터 저장
-----
```sql
INSERT INTO TABLE_NAME Sub-Query;
```

1. 다음과 같이 EMP02 TABLE과 구조가 존재한다고 하자.
```sql
CREATE TABLE EMP02
AS
SELECT EMPNO, ENAME, JOB
FROM EMP
WHERE 1=0;
```

```sql
EMPNO NUMBER(4, 0)
ENAME VARCHAR(10)
JOB VARCHAR2(9)
```

2. Sub-Query를 통한 데이터 저장 예
```sql
INSERT INTO EMP02(EMPNO, ENAME, JOB)
SELECT EMPNO, ENAME, JOB FROM EMP;
```
   - 1:1로 매칭됨

   - 동일하게, 모든 컬럼 항목에 대해 삽입할 경우에는 생략이 가능
```sql
INSERT INTO EMP01
SELECT EMPNO, ENAME, JOB FROM EMP;
```


```sql
INSERT ALL
INTO TABLE_NAME(COL_NAME) VALUES(COL_NAME)
INTO TABLE_NAME(COL_NAME) VALUES(COL_NAME);
```
3. 다음과 같이 EMP03 테이블과 EMP04 테이블이 존재한다고 하자.
```sql
CREATE TABLE EMP03
AS
SELECT EMPNO, ENAME, JOB FROM EMP WHERE 1 = 0;
```
```sql
CREATE TABLE EMP04
AS
SELECT EMPNO, ENAME, HIREDATE FROME EMP WHERE 1=0;
```

```sql
INSERT ALL
INTO EMP03(EMPNO, ENAME, JOB) VALUES(EMPNO, ENMAE, JOB)
INTO EMP04(EMPNO, ENAME, HIREDATE) VALUES(EMPNO, ENAME, HRIEDATE)
SELECT EMPNO, ENAME, JOB, HIREDAT FROM EMP;
```
   - 서브쿼리를 통해 가져올 수 있고, 서브쿼리에 해당된 열에 대해 각각 삽입이 가능

-----
### INSEERT 예제
-----
1. 사원번호, 이름, 급여를 저장할 수 있는 빈 테이블을 만든다.
```sql
CREATE TABLE EMP05
AS
SELECT EMPNO, ENAME, SAL, FROM EMP WHERE 1=0;
```

2. 급여가 1500 이상인 사원들의 사원번호, 이름, 급여를 저장한다.
```sql
INSERT INTO EMP05(EMPNO, ENAME, SAL)
WHERE EMPNO, ENAME, SAL FROM EMP WHERE SAL >= 1500;
```

3. 사원번호, 이름, 부서명을 저장할 수 있는 빈 테이블을 만든다.
```sql
CREATE TABLE EMP06
AS
SELECT E1.EMPNO, E1.ENAME, D1.DNAME FROM EMP E1 DEPT D1 WHERE (E1.DEPTNO = D1.DEPTNO) AND (1 = 0);
```

4. DALLAS 지역에 근무하고 있는 사원들의 사원번호, 이름, 부서명을 저장한다.
```sql
INSERT INTO EMP06(EMPNO, ENAME, DNAME)
SELECT E1.EMPNO, E1.ENAME D1.DANME FROM EMP E1 DEPT D1 WHERE (E1.DEPTNO = D1.DEPTNO) AND (D1.LOC = 'DALLAS');  
