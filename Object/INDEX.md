-----
### 인덱스 (Index)
-----
1. 데이터베이스에서 검색 속도를 향상시키기 위해 사용하는 데이터베이스 객체
2. 인덱스의 장점
   - 검색 속도가 향상
   - 시스템의 부하를 줄여 성능 향상
3. 인덱스의 단점
   - 추가적인 기억 공간 필요
   - 인덱스 생성 시간이 오래 걸림
   - INSERT, UPDATE, DELETE와 같은 변경 작업이 빈번하게 발생하면, 오히려 성능 저하 발생
4. 일반적으로 Primary Key를 인덱스로 설정
5. 인덱스의 분류
   - 인덱스 구성 컬럼 개수에 따른 분류 : 단일 인덱스 / 결합 인덱스
   - 유일성 여부에 따른 분류 : UNIQUE 인덱스, NON-UNIQUE 인덱스 
   - 인덱스 내부 구조에 따른 분류 : B-TREE 인덱스, 비트맵 인덱스, 함수 기반 인덱스
6. 인덱스는 테이블에 존재하는 한 개 이상 컬럼으로 만들 수 있음

-----
### 인덱스 (Index) 생성 및 조회
-----
1. 형식
```sql
CREATE [UNIQUE] INDEX [스키마명.]인덱스명
ON [스키마명.]테이블명 (컬럼1, 컬럼2, ...);
```

2. TEST_TABLE100 테이블을 생성한다.
```sql
CREATE TABLE TEST_TABLE100 (
DATA1 NUMBER CONSTRAINT TEST_TABLE100_DATA1_PK PRIMARY KEY,
DATA2 NUMBER NOT NULL
);
```

3. EMP01 테이블에 대해 EMP01_IDX 인덱스를 생성하는데, EMP01 테이블의 ENAME을 인덱스로 생성한다.
```sql
CREATE INDEX EMP01_IDX
ON EMP01(ENAME);
```

4. 인덱스 조회 방법
```sql
SELECT INDEX_NAME, TABLE_NAME, COLUMN_NAME
FROM USER_IND_COLUMNS;
[WHERE TABLE_NAME = '테이블명';]
```
<div align = "center">
<img src="https://github.com/sooyounghan/Web/assets/34672301/d8ca46f3-acaf-4fdc-a3fc-e41fa7475b67">
</div>   

  - Oracle에서는 PK에 대해 자동적으로 인덱스를 생성
  - 그러므로, PK의 값으로 검색하게 되면 검색 속도가 향상

-----
### 인덱스 (Index) 생성 시 고려할 사항
-----
1. 일반적으로 테이블 전체 ROW의 15%의 데이터를 조회할 떄 인덱스를 생성 (데이터 분포 정도에 따라 달라짐)
2. 테이블 건수가 적다면, 굳이 인덱스를 만들 필요가 없음
   - 스캔(Scan) : 데이터 추출을 위해 테이블이나 인덱스를 탐색하는 것
   - 테이블 건수가 적으면 인덱스를 경유하기보다 테이블 전체 스캔이 빠름
      
3. 데이터의 유일성 정도가 좋거나 범위가 넓은 값을 가진 컬럼을 인덱스로 만드는 것이 좋음
4. 결합 인덱스를 만들 때는, 컬럼의 순서가 중요
   - 자주 사용되는 컬럼을 순서상 앞에 두는 것이 좋음
      
5. 테이블에 만들 수 있는 인덱스 수는 제한이 없으나, 오히려 많이 만들면 성능 부하 발생
 
