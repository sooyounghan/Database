-----
### 뷰(VIEW)
-----
1. 데이터베이스가 제공하는 가상의 테이블
2. 뷰를 사용하면, 복잡한 쿼리문을 대신할 수 있기 때문에 개발의 용이성 제공
3. 뷰를 만들 때 사용한 쿼리문을 저장하는 것이며, 뷰를 조회할 때는 뷰를 만들 때 사용한 쿼리문이 동작
4. 즉, 하나 이상의 테이블이나 다른 뷰의 데이터를 볼 수 있게 하는 데이터베이스 객체

-----
### 뷰(VIEW) 생성 권한 설정
-----
1. 일반 계정의 경우 뷰를 생성할 수 있는 권한이 없으므로 뷰 생성 권한을 설정해야함
2. 형식
```sql
GRANT CREATE VIEW TO 계정이름;
```

-----
### 뷰(VIEW) 생성
-----
1. 형식
```sql
CREATE [OR REPLACE] VIEW [스키마명.]뷰명
AS
SELECT 문장(서브쿼리);
```

2. 사원의 사원번호, 이름, 급여, 근무부서이름, 근무지역을 가지고 이는 생성한다. 그리고, 이 쿼리문에 대한 뷰를 생성한다.
```sql
CREATE VIEW EMP_DEPT_VIEW
AS
SELECT E.EMPNO, E.ENAME, E.SAL, D.DNAME, D.LOC 
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DEPTNO;
```


-----
### 뷰(View) 조회
-----
1. 형식
```sql
SELECT 컬럼명,...
FROM 뷰명;
[WHERE 조건절]
```

2. 위에서 생성한 EMP_DEPT_VIEW를 조회한다.
```sql
SELECT *
FROM EMP_DEPT_VIEW;
```
  - 뷰를 조회하면, 뷰 생성 시 동작했던 서브쿼리가 동작 (즉, 이 쿼리문이 저장되어 있는 것)

-----
### 뷰(View) 삭제
-----
1. 형식
```sql
DROP VIEW [스키마명.]뷰명;
```

2. 뷰를 삭제하더라도 실제 데이터는 삭제되지 않음

-----
### 뷰(View) 예제
-----
1. DEPT100 테이블을 생성한다. DEPT 테이블의 정보를 복사한다. EMP100 테이블은 EMP 테이블을 복사한다.
```sql
CREATE TABLE DEPT100
AS
SELECT * FROM DEPT;
```
```sql
CREATE TABLE EMP100
AS
SELECT * FROM EMP;
```

2. DEPT100 테이블과 EMP100 테이블에 대한 뷰를 생성한다.
```sql
CREATE VIEW EMP100_DEPT100_VIEW
AS
SELECT E.EMPNO, E.ENAME, E.SAL, D.DNAME, D.LOC
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DPETNO;
```

3. EMP100_DEPT100_VIEW 를 조회한다.
```sql
SELECT *
FROM EMP100_DEPT100_VIEW;
```

4. 원본 테이블에 데이터를 삽입한다.
```sql
INSERT INTO EMP100(EMPNOM, ENAME, SAL, DEPTNO)
VALUES(5000, '홍길동', 2000, 10);
```
  - 뷰를 통해 정보를 확인하면, 데이터가 동일하게 추가된다.

5. 뷰를 통해 데이터를 삽입한다.
```sql
INSERT INTO EMP100_DEPT100_VIEW(EMPNOM, ENAME, SAL)
VALUES(6000, '김길동', 2000);
```
  - 키-보존된 것이 아닌 테이블로 대응한 열을 수정할 수 없다는 에러 발생
  - 두 개의 테이블을 JOIN해서 가져왔으므로, 오류 발생 (JOIN을 통해 생성된 뷰를 통해 데이터 삽입 불가)
  - JOIN하지 않은 테이블로 생성한 뷰의 경우에는 뷰를 통해 데이터 삽입 가능
