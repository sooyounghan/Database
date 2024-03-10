-----
### 제약조건 (Constraint)
-----
1. 테이블의 데이터를 저장 혹은 수정할 때 컬럼의 값에 대한 조건을 설정하는 것
2. 설정된 조건에 위배되는 값을 컬럼에 저장할 수 없으며, 데이터 무결성을 위한 구문
3. 컬럼에 대한 속성 형태로 정의하지만, 데이터베이스 객체 중 하나임
4. 제약조건명을 설정하지 않으면, 명확하게 발생한 제약조건명을 확인하기 힘드므로, 제약조건명을 사용하는 것이 관례

-----
### 데이터의 무결성(Data Integrity)
-----
: 데이터의 정확성과 일관성을 유지하는 것

-----
### NOT NULL
-----
1. 컬럼에 NULL을 허용하지 않는 것
2. 즉, 해당 컬럼에는 반드시 데이터를 입력해야함
3. 즉, 반드시 값이 있어야 하는 컬럼에는 NOT NULL 제약 조건을 만들어 사용
```sql
컬럼명 데이터타입(크기) NOT NULL
OR
COSTRAINT 제약조건명 NOT NULL(컬럼명, ...)
```

4. TEST_TABLE1을 생성한다.
   - DATA1 컬럼은 NULL을 허용하는 NUMBER형
   - DATA2 컬럼은 NULL을 허용하지 않는 NUMBER형
     
```sql
CREATE TABLE TEST_TABLE1 (
DATA1 NUMBER,
DATA2 NUMBER NOT NULL
);
```

5. TEST_TABLE1에 DATA1, DATA2 값을 삽입한다.
```sql
INSERT INTO TEST_TABLE1(DATA1, DATA2)
VALUES(100, 101);
```
  - 100, 101 값이 삽입이 잘 되어있음

```sql
INSERT INTO TEST_TABLE1(DATA1)
VALUES(100);
```
  - 오류가 발생 (DATA2는 NOT NULL로 설정)

```sql
INSERT INTO TEST_TABLE1(DATA2)
VALUES(201);
```
  - DATA1은 NULL 값 허용, DATA2에는 201 값 삽입

-----
### UNIQUE
-----
1. 중복된 값을 허용되지 않음 (NULL값은 무한대로 적용 가능)
```sql
컬럼명 데이터타입(크기) UNIQUE
OR
CONSTAINTS 제약조건명 UNIQUE(컬럼명, ...)
```

2. TEST_TABLE2를 생성한다.
   - CONSTRAINTS 제약조건명 UNIQUE
```sql
CREATE TABLE TEST_TABLE2 (
DATA1 NUMBER,
DATA2 NUMBER CONSTRAINTS TEST_TABLE2_DATA2_UK UNIQUE
);
```
3. 데이터 100과 101을, 200과 201을 삽입한다
```sql
INSERT INO TEST_TABLE2(DATA1, DATA2)
VALUES (100, 101)
VALUES (200, 201);
```

4. 300, 201 값을 삽입한다.
```sql
INSERT INO TEST_TABLE2(DATA1, DATA2)
VALUES (300, 201);
```
  - 오류 발생 : 무결성 제약조건(스키마명.TEST_TABLE2_DATA2_UK)에 위배됩니다.

5. (200, NULL)과 200만을 넣는다.
```sql
INSERT INO TEST_TABLE2(DATA1, DATA2)
VALUES (300, NULL);
```
```sql
INSERT INO TEST_TABLE2(DATA1)
VALUES (300);
```
  - NULL 값은 문제가 되지 않으며, 무한대로 삽입 가능

-----
### PK(Primary Key) : UNIQUE + NOT NULL
-----
1. 중복된 값을 허용하지 않으며, NULL 값을 허용하지 않음
2. ROW를 구분하기 위한 유일한 값을 저장하기 위해 사용
3. 테이블 당 1개의 기본키만 생성 가능
```sql
컬럼명 데이터타입(크기) PRIMARY KEY
OR
CONSTRAINTS 제약조건명 PRIMARY KEY(컬럼명, ...)
```

4. TEST_TABLE3을 생성한다.
   - DATA2는 기본키
```sql
CREATE TABLE TEST_TABLE3 (
DATA1 NUMBER,
DATA2 NUMBER CONSTRAINTS TEST_TABLE3_DATA2_PK PRIMARY KEY
);
```

5. 데이터 100과 101을 삽입한다
```sql
INSERT INO TEST_TABLE2(DATA1, DATA2)
VALUES (100, 101);
```

6. 다시 한 번 5번의 값을 삽입한다.
```sql
INSERT INO TEST_TABLE2(DATA1, DATA2)
VALUES (100, 101);
```
  - 오류 발생 : 무결성 제약 조건(TEST_TABLE3_DATA2_PK)에 위배됩니다.
  - DATA2는 PRIMARY KEY로서, 중복된 값이 삽입되려고 하므로 오류 발생

```sql
INSERT INO TEST_TABLE2(DATA1, DATA2)
VALUES (100, NULL);
```
  - 오류 발생 : 무결성 제약 조건(TEST_TABLE3_DATA2_PK)에 위배됩니다.
  - DATA2는 PRIMARY KEY로서, NULL 값 불가


-----
### FK(Primary Key)
-----
1. 다른 테이블 혹은 같은 테이블의 컬럼을 참조하는 제약 조건
2. 참조하는 컬럼에 저장되어 있는 값만 컬럼에 저장 가능
3. 일반적으로 PRIMARY KEY 제약 조건이 설정된 컬럼을 참조
4. NULL 값은 가능하며, NULL이 아닌 값은 반드시 PRIMARY KEY에 설정된 값이어야 함
```sql
CONSTRAINTS 외래키명 FOREIGN KEY(컬럼명, ...)
REFERENCES 참조 테이블(참조 테이블 컬럼명, ...)
```

4. 외래키에 대한 제약사항
   - 반드시 참조하는 테이블이 먼저 생성
   - 참조키가 참조 테이블의 기본키로 만들어져야 함
   - 외래키에 사용할 수 있는 컬럼 수는 최대 32개
   - 여러 컬럼을 외래키로 만들려면, 참조하는 컬럼과 외래키 컬럼의 수는 같아야 함

5. 테이블 TEST_TABLE4 생성한다.
```sql
CREATE TABLE TEST_TABLE4 (
DATA1 NUMBER CONSTRAINTS TEST_TABLE1_DATA4_PK PRIMARY KEY,
DATA2 NUMBER
);
```

6. (100, 101), (200, 201) 값을 삽입한다.
```sql
INSERT INO TEST_TABLE4(DATA1, DATA2)
VALUES (100, 101)
VALUES (200, 201);
```

7. 테이블 TEST_TABLE5를 생성한다.
```sql
CREATE TABLE TEST_TABLE5 (
DATA3 NUMBER,
DATA4 NUMBER CONSTRAINT TEST_TABLE5_DATA4_FK REFERENCES TEST_TABLE4(DATA1)
);
```
  - DATA4는 FK로서, TEST_TABLE4의 PK 키를 참조

8. 다음 값을 삽입한다.
```sql
INSERT INO TEST_TABLE5(DATA1, DATA2)
VALUES (1, 100)
VALUES (2, 100)
VALUES (3, 200)
VALUES (4, 200);
```
  - FK는 중복 허용이 가능

```sql
INSERT INO TEST_TABLE5(DATA3, DATA4)
VALUES (5, NULL);
```
```sql
INSERT INO TEST_TABLE5
VALUES (6);
```
  - NULL 삽입 가능, 생략해도 NULL값 삽입

```sql
INSERT INO TEST_TABLE5(DATA3, DATA4)
VALUES (7, 300);
```
  - 오류 발생 : 무결성 제약조건(TEST_TABLE5_DATA4_FK)이 위배되었습니다. 부모 키가 없습니다.

-----
### CHECK
-----
1. 조건에 만족할 경우 컬럼에 저장할 수 있도록 하는 제약조건을 지정
2. 만족하지 않으면 오류를 발생
```sql
CONSTRAINTS 제약조건명 CHECK(제약 조건)
```

3. 테이블 TEST_TABLE6를 생성한다.
   - DATA1 컬럼에는 1부터 10까지만 삽입되도록 범위를 설정하는 제약조건
   - DATA2 컬럼에는 10, 20, 30만 삽입되도록 값을 설정하는 제약 조건
```sql
CREATE TABLE TEST_TABLE6 (
DATA1 NUMBER CONSTRAINTS TEST_TABLE6_DATA1_CK CHECK(DATA1 BETWEEN 1 AND 10),
DATA2 NUMBER CONSTRAINTS TEST_TABLE6_DATA2_CK CHECK(DATA2 IN (10, 20, 30)
);
```
4. 데이터를 삽입한다.
```sql
INSERT INO TEST_TABLE6(DATA1, DATA2)
VALUES (1, 10)
VALUES (2, 20);
```
  - 오류 없음

```sql
INSERT INO TEST_TABLE6(DATA1, DATA2)
VALUES (20, 10)
VALUES (5, 100);
```
  - 첫 번째 구문 오류 발생 : 무결성 제약조건(TEST_TABLE6_DATA1_CK)이 위배되었습니다.
  - 두 번째 구문 오류 발생 : 무결성 제약조건(TEST_TABLE6_DATA2_CK)이 위배되었습니다.

-----
### DEFAULT
-----
1. 제약조건에 포함되지 않지만, 컬럼의 디폴트 값을 명시하는데 사용
2. DEFAULT 디폴트 값을 기술하면, 자동으로 디폴트 값이 생성

-----
### 테이블 레벨 제약 조건
-----
1. 제약조건 설정 시, 각 컬럼 마다 지정할 수 있지만 하단 부분에 몰아서 지정 가능
2. 컬럼 명 옆에 기술하는 것을 컬럼 레벨, 하단에 몰아서 기술하는 것을 테이블 레벨 제약 조건

3. TEST_TABLE10을 생성 (컬럼 레벨)
```sql
CREATE TABLE TEST_TABLE10 (
DATA1 NUMBER CONSTRAINT TEST_TABLE10_DATA1_PK PRIMARY KEY,
DATA2 NUMBER NOT NULL CONSTRAINTS TEST_TABLE10_DATA2_UK UNIQUE,
DATA3 NUMBER NOT NULL CONSTRAINTS TEST_TABLE10_DATA3_FK REFERENECES EMP(EMPNO),
DATA4 NUMBER NOT NULL CONSTRAINTS TEST_TABLE10_DATA4_CK CHECK(DATA4 BETWEEN 1 AND 10),
DATA5 NUMBER NOT NULL CONSTRAINTS TEST_TABLE10_DATA5_CK CHECK(DATA5 IN(10, 20, 30))
);
```
   - 컬럼명 옆에 바로 제약 조건 명시
     
4. TEST_TABLE11을 생성 (테이블 레벨)
```sql
CREATE TABLE TEST_TABLE11 (
DATA1 NUMBER NOT NULL,
DATA2 NUMBER NOT NULL,
DATA3 NUMBER NOT NULL,
DATA4 NUMBER NOT NULL,
DATA5 NUMBER NOT NULL,

CONSTRAINTS TEST_TABLE11_DATA1_PK PRIMARY KEY(DATA1),
CONSTRAINTS TEST_TABLE12_DATA2_UK UNIQUE(DATA2),
CONSTRAINTS TEST_TABLE13_DATA3_FK FOREIGN KEY(DATA3) REFERENCES EMP(EMPNO),
CONSTRAINTS TEST_TABLE14_DATA4_CK CHECK(DATA4 BETWEEN 1 AND 10),
CONSTRAINTS TEST_TABLE15_DATA5_CK CHECK(DATA5 IN(10, 20, 30))
);
```
   - 컬럼명 옆이 아닌 컬럼명 선언 후, 하단에 몰아서 작성

-----
### 복합키 (테이블 레벨로만 가능)
-----
1. 테이블 레벨 제약 조건을 설정할 때 하나 이상의 컬럼을 하나의 PRIMARY KEY로 묶어서 사용 가능
2. 복합키의 경우 각 컬럼에 중복된 데이터가 허용이 되지만, 하나의 ROW에 모든 복합키 컬럼이 중복되는 것은 허용하지 않음

3. TEST_TABLE12을 생성 (테이블 레벨)
```sql
CREATE TABLE TEST_TABLE12 (
DATA1 NUMBER,
DATA2 NUMBER,

CONSTRAINTS TEST_TABLE12_COMBO_PK PRIMARY KEY(DATA1, DATA2)
);
```

4. (100, 200) 값을 입력
```sql
INSERT INTO TEST_TABLE12(DATA1, DATA2)
VALUES(100, 200);
```

5. (100, 300) / (200, 400) 값을 입력 [한 키만 중복 : 문제 없음(ROW 단위에는 문제 없음]
```sql
INSERT INTO TEST_TABLE12(DATA1, DATA2)
VALUES(100, 300);
VALUES(200, 400);
```

4. (100, 200) 값을 다시 입력
```sql
INSERT INTO TEST_TABLE12(DATA1, DATA2)
VALUES(100, 200);
```
   - 오류 발생 - 무결성 제약조건(TEST_TABLE12_COMBO_PK)에 위배됩니다.
   - 복합키로 설정된 모든 키가 중복이면, 오류 발생
   - 또한, NULL 값은 PK이므로 오류 발생
