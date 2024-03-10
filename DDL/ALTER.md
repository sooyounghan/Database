-----
### ALTER : 제약조건 추가 및 삭제
-----
1. 테이블을 생성한 후 제약 조건을 추가하거나 제거하고 싶다면 ALTER 구문 이용
2. 형식
```sql
ALTER TABLE [스키마.]테이블명
ADD CONSTARINTS 제약조건명 제약조건

ALTER TABLE [스키마.]테이블명
DROP CONSTRAINTS 제약조건명;
```

3. 제약조건으로 TEST_TABLE에 DATA1에 기본키를 추가한다.
```sql
ALTER TABLE TEST_TABLE
ADD CONSTRAINTS TEST_TABLE_DATA1_PK PRIMARY KEY(DATA1);
```

4. 제약조건으로 TEST_TABLE에 기본키인 DATA1의 기본키 설정을 삭제한다.
```sql
ALTER TABLE TEST_TABLE
DROP CONSTRAINTS TEST_TABLE_DATA1_PK;
```

-----
### 컬럼 추가 및 삭제
-----
1. 형식
```sql
ALTER TABLE TEST_TABLE
ADD COLUMN 컬럼명 데이터타입;
```

```sql
ALTER TABLE TEST_TABLE
DROP COLUMN 컬럼명;
```

2. TEST_TABLE에 DATA1 컬럼을 추가하고, 삭제한다.
```sql
ALTER TABLE TEST_TABLE
ADD COLUMN DATA1 NUMBER;
```
```sql
ALTER TABLE TEST_TABLE
DROP COLUMN;
```
-----
### 컬럼명 변경
-----
1. 형식
```sql
ALTER TABLE [스키마.]테이블명
RENAME COLUMN 변경전컬럼명 TO 변경후컬럼명;
```

2. TEST_TABLE에 있는 DATA1 컬럼을 DATA2 컬럼으로 이름을 변경
```sql
ALTER TABLE TEST_TABLE
RENAME COLUMN DATA1 TO DATA2;
```

-----
### 컬럼타입 변경
-----
1. 형식
```sql
ALTER TABLE [스키마.]테이블명
MODIFY 컬럼명 데이터타입;
```

2. TEST_TABLE에 있는 DATA1의 컬럼의 타입을 VARCHAR2(30)으로 변경
```sql
ALTER TABLE [스키마.]테이블명
MODIFY DATA1 VARCHAR2(30);
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
