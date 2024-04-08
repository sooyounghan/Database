-----
### 테이블 구조 변경하기 : 컬럼 추가 및 삭제
-----
1. 컬럼 추가
```sql
ALTER TABLE [스키마.]테이블명
ADD COLUMN 컬럼명 데이터타입 [제약조건];
```

2. 컬럼 삭제
```sql
ALTER TABLE [스키마.]테이블명
DROP COLUMN 컬럼명;
```

3. 테이블 삭제
```sql
DROP TABLE 테이블명 [CASCADE CONSTRAINTS];
```
: CASCADE CONSTRAINTS를 붙이면, 삭제할 테이블의 기본키와 UNIQUE 키를 참조하는 참조 무결성 제약조건도 자동으로 삭제

-----
### 컬럼명 변경
-----
1. 컬럼명 변경
```sql
ALTER TABLE [스키마.]테이블명
RENAME COLUMN 변경전컬럼명 TO 변경후컬럼명;
```

2. 테이블 이름 변경
```sql
ALTER TABLE [스키마.]테이블명
RENAME TO 테이블명;
```

-----
### 컬럼타입 변경
-----
1. 형식
```sql
ALTER TABLE [스키마.]테이블명
MODIFY 컬럼명 데이터타입 [제약조건];
```

-----
### 예제
-----
1. TEST_TABLE1 TABLE을 생성한다.
```sql
CREATE TABLE TEST_TABLE1 (
DATA1 NUMBER NOT NULL,
DATA2 NUMBER NOT NULL
);
```

2. TEST_TABLE1에 DATA3 컬럼을 추가하되, 제약조건은 NOT NULL로 설정한다.
```sql
ALTER TABLE TEST_TABLE1
ADD COLUMN (DATA3 NUMBER NOT NULL);
```

3. TEST_TABLE1에 있는 DATA3 컬럼에 대해 데이터 타입을 VARCHAR2(100) 타입으로 변경한다.
```sql
ALTER TABLE TEST_TABLE1
MODIFY (DATA3 VARCHAR2(100));
```

4. TEST_TABLE1의 이름을 TEST_TABLE2로 변경한다.
```sql
ALTER TABLE TEST_TABLE1
RENAME TO TEST_TABLE2;
```

5. TEST_TABLE2의 DATA3 컬럼의 이름을 DATA4로 변경한다
```sql
ALTER TABLE TEST_TABLE2
RENAME COLUMN DATA3 TO DATA4;
```

6. TEST_TABLE2의 DATA4 컬럼을 삭제한다.
```sql
ALTER TABLE TEST_TABLE2
DROP COLUMN DATA4;
```

7. TEST_TABLE2 TABLE를 삭제한다.
```sql
DROP TABLE TEST_TABLE2;
```

-----
### 테이블 복사
-----
1. 형식
```sql
CREATE TABLE [스키마.]테이블명
AS
SELECT 컬럼1, 컬럼2,...
FROM 복사할 테이블명;
```

2. EMP TABLE의 사원정보, 사원이름, 급여 정보를 EMP01 테이블에 복사한다.
```sql
CREATE TABLE EMP01
AS
SELECT EMPNO, ENAME, SAL
FROM EMP;
```
