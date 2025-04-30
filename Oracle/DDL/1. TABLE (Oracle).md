-----
### 데이터베이스 객체
-----
1. 데이터베이스 내 존재하는 논리적 저장 구조
2. 종류
   - TABLE : 데이터를 담고 있는 객체
   - VIEW : 하나 이상의 테이블을 연결해 마치 테이블인 것처럼 사용하는 객체
   - INDEX : 테이블에 있는 데이터를 빠르게 찾기 위한 객체
   - SYNONYM : 데이터베이스 객체에 대한 별칭을 부여한 객체
   - FUNCTION : 특정 연산을 하고 값을 반환하는 객체
   - PROCEDUER : 함수와 비슷하지만 값을 반환하지 않는 객체
   - PACKAGE : 용도에 맞게 함수나 프로시저를 하나로 묶어 놓은 객체

-----
### 테이블
-----
1. 데이터를 넣고 수정하고 삭제하는, 즉 데이터를 담고 있는 객체
2. DBMS에서 가장 기본적 개체로 ROW와 COLUMN으로 구성된 2차원 형태(표) 객체

-----
### CREATE TABLE
-----
1. Oracle
```sql
CREATE TABLE [스키마.]테이블명 (
  컬럼명1 DATATYPE(컬럼1_자료형)(크기) [CONSTRAINT 제약조건명] [제약조건 / [NULL, NOT NULL]]
  컬럼명2 DATATYPE(컬럼2_자료형)(크기) [CONSTRAINT 제약조건명] [제약조건 / [NULL, NOT NULL]]
  컬럼명3 DATATYPE(컬럼3_자료형)(크기) [제약조건] DEFAULT 기본값 
...) [TABLESPACE 테이블스페이스명];
```

2. MySQL
```sql
CREATE TABLE [스키마.]테이블명 (
  컬럼명1 DATATYPE(컬럼1_자료형)(크기) [CONSTRAINT 제약조건명] [제약조건 / [NULL, NOT NULL]]
  컬럼명2 DATATYPE(컬럼2_자료형)(크기) [CONSTRAINT 제약조건명] [제약조건 / [NULL, NOT NULL]]
  컬럼명3 DATATYPE(컬럼3_자료형)(크기) [제약조건] DEFAULT 기본값 
...) ENGINE = InnoDB DEFAULT CHARSET = UTF-8;
```

-----
### 자료형 (DataType)
-----
1. 컬럼이 저장되는 데이터 유형
2. 기본 데이터 타입(원시 데이터 타입)과 사용자 정의 데이터 타입으로 구분
3. 문자 데이터 타입 (문자나 문자열 데이터)
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/d6acf32c-dac4-4f1b-836d-760eabcc62e2">
</div>   

  - 가변길이 : 실제 입력된 데이터 길이에 따라 크기가 변해 정해지는 값
  - byte : byte 크기 / char : 글자 수
  - char : 데이터 편차가 심하지 않은 데이터에 사용
  - varchar2 : 데이터 편차가 심한 데이터에 사용

        예) VARCHAR2(10)으로 선언 : 10byte까지 데이터 입력 가능
            - 'bac'라고 입력 : 실제 컬럼 길이는 3byte
        예) CHAR(10) : 10byte까지 데이터 입력 가능
            - 'bac'라고 입력 : 실제 데이터 길이 10byte

4. 숫자 데이터 타입
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/62fbaf0f-1a6b-4223-8bba-cd13bbbcf3e3">
</div>   

  - NUMBER, NUMBER(*) : 디폴트 값인 38이 적용되어 최대 크기인 22byte 차지 (최고 40자리)
  - NUMBER(p(precision, 정밀도)), s(scale, 자릿수))

         A. p : 소수점 기준 모든 유효숫자 자릿수 (p보다 명시한 것보다 큰 값 입력 : 오류)
         B. s : 양수면 소주점 이하, 음수면 소수점 이상 유효숫자 자릿수
         C. s에 명시한 숫자 이상 숫자를 입력하면, s에 명시한 숫자로 반올림 처리
         D. s가 음수면 소수점 기준 왼쪽 자릿수만큼 반올림
         E. s > p : p는 소수점 이하 유효숫자 자릿수

  - 유형
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/76f949a5-7608-4451-bec8-49972c767f7f">
</div>

5. 날짜 데이터 타입
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/526438e3-9dce-49d7-aa94-7af2606bbb4f">
</div>

6. LOB 데이터 타입
   - Large Object의 약자로 대용량 데이터를 저장할 수 있는 데이터 타입
   - 문자형 대용량 데이터는 CLOB나 NCLOB, 그래픽/이미지/동영상 등 데이터는 BLOB
   - BFILE은 데이터베이스 외부 파일에 대한 로케이터를 저장 (읽기만 가능)
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/f74261eb-569c-42d6-b66b-ed9bd6af25ce">
</div>

7. NULL
   - '값이 없음'을 의미하며, 테이블 생성 시 컬럼 속성에 기술
   - 디폴트 값이 NULL이므로 별도로 지정하지 않으면 컬럼은 NULL 허용
   - NULL을 허용하지 않으려면, NOT NULL 구문 명시

-----
### 테이블 제약 조건 조회
-----
* user_constraints는 제약조건을 담는 테이블 객체 (소문자 주의)
  
1. 접속한 계정에 대한 테이블의 제약 조건 조회
```sql
SELECT *
FROM user_constraints;
```

2. 특정 테이블의 제약 조건 조회
```sql
SELECT *
FROM user_constraints
WHERE TABLE_NAME = '테이블명';
```

-----
### 테이블 생성 예제
-----
1. 다음과 같은 정보를 저장하기 위한 테이블을 만든다.
2. 학생 정보, 학생 이름, 학생 나이, 학생 국어 점수, 영어 점수, 수학 점수
```sql
CREATE TABLE STU_TABLE (
STU_ID NUMBER,
STU_NAME CHAR(10),
STU_AGE NUMBER,
STU_KOR NUMBER,
STU_ENG NUMBER,
STU_MATH NUMBER
);
```

   - 위 테이블에 1111, 홍길동, 30, 100, 80, 70 점의 값을 삽입한다.
```sql
INSERT INTO STU_TABLE(STU_ID, STU_NAME, STU_AGE, STU_KOR, STU_ENG, STU_MATH)
VALUES(1111, '홍길동', 30, 100, 80, 70);
```

3. 숫자형 테이블 생성
```sql
CREATE TABLE NUMBER_TABLE (
NUMBER1 NUMBER,
NUMBER2 NUMBER(3),
NUMBER3 NUMBER(5, 2)
```

```
INSERT INTO NUMBER_TABLE(NUMBER1) VALUES(10000);
INSERT INTO NUMBER_TABLE(NUMBER2) VALUES(10000);
INSERT INTO NUMBER_TABLE(NUMBER1) VALUES(100.111111);
```
  -  1번은 삽입 가능
  -  2번은 삽입 불가 (NUMBER(3)이므로 총 3자리만 가능)
  -  3번은 삽입 가능 (NUMBER(5, 2)이므로 총 5자리만 가능하지만 소수점 자리는 2자리까지 가능)

-----
### 서브쿼리를 통한 테이블 생성 : 단, 제약조건 까지 테이블이 복사되지 않음
-----
```sql
CREATE TABLE 테이블명
AS
SUB_QUERY;
```
1. EMP 테이블을 복제한 EMP01 테이블을 생성
```sql
CREATE TABLE EMP01
AS
SELECT * FROM EMP;
```

2. EMP 테이블에서 사원번호, 이름, 급여 정보를 가지고 있는 테이블 EMP02 생성
```sql
CREATE TABLE EMP02
AS
SELECT EMPNO, ENAME, SAL FROM EMP;
```

3. 30번 부서에 근무하고 있는 사원들의 사원번호, 이름, 근무부서 이름을 가지는 EMP03 테이블 생성
```sql
CREATE TABLE EMP03
AS
SELECT E.EMPNO, E.ENAME, D.DNAME
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DEPTNO;
```

4. 각 부서별 급여 총합, 평균, 최고액, 최저액, 사원수를 가지고 있는 EMP04 테이블 생성
```sql
CREATE TABLE EMP04
AS
SELECT SUM(SAL) AS SUM, AVG(SAL) AS AVG, MAX(SAL) AS MAX, MIN(SAL) AS MIN, COUNT(SAL) AS COUNT
FROM EMP 
GROUP BY DEPTNO;
```
  - 그룹함수로 가져온 데이터에 대해 이름이 없기 때문에, 이름을 정해야함

5. 회원의 번호, 아이디, 비밀번호, 이름, 성별, 등록일을 가지는 MEMBER 테이블을 생성
   - 아이디와 비밀번호, 이름은 NULL 값이 아니여야 한다.
   - 아이디는 유일해야한다. (다중 제약조건)
```sql
CREATE TABLE MEMBER (
MEMBER_NO NUMBER(6) CONSTRAINT MEMBER_MEMBER_NO_PK PRIMARY KEY,
MEMBER_ID VARCHAR2(20) CONSTRAINT MEMBER_MEMBER_NN NOT NULL,
MEMBER_PASSWORD VARCHAR2(20) CONSTRAINT MEMBER_MEMBER_PASSWORD_NN NOT NULL,
MEMBER_MNAME VARCHAR2(45) CONSTRAINT MEMBER_MEMBER_MANME_NN NOT NULL,
MEMBER_GENDER CHAR(1) CONSTRAINT MEMBER_MEMBER_GENDER_CK CHECK(MEMBER_GENDER IN ('M', 'F', 'Z')),
MEMBER_REGDATE DATE DEFAULT SYSDATE,

CONSTRAINT MEMBER_MEMBER_ID_UK UNIQUE(MEMBER_ID)
);
```

   - 제약 조건이 2개 이상 이라면, 한 개는 테이블 레벨 / 한 개는 컬럼 레벨에서도 설정 가능
   - NULL 제약 조건은 컬럼 레벨에서만 가능
