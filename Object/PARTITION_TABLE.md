-----
### 파티션 테이블 (Partition Table)
-----
1. Partition : 테이블에 있는 특정 컬럼 값을 기준으로 데이터를 분할해 저장해 놓는 것
2. 논리적 테이블은 1개지만, 물리적으로는 분할한 만큼 파티션이 만들어져 입력되는 컬럼 값에 따라 분할된 파티션별로 데이터가 저장
3. 대용량 테이블의 경우 데이터 조회 시 효율성과 성능을 높이기 위함

-----
### 파티션 테이블 (Partition Table) 예제
-----
1. 예를 들어, Sample Schema에는 매출정보가 있는 SALES 테이블에는 총 91만여건의 데이터가 존재
2. 이 테이블에서 특정 데이터를 찾으려면 91만건의 조회가 필요함
3. 이 때, 테이블에 있는 판매일자(SALES_DATE)와 판매월(SALES_MONTH) 컬럼을 이용해 조회할 때 성능을 높여보자.
4. 즉, 판매월별로 데이터를 분할해 놓고, 데이터 조회 시 특정 월을 조건으로 걸면 전체 데이터를 뒤져보지 않아도 가능

   - 파티션이 없는 경우 : 91만여건의 데이터 조회
   - 파티션이 있는 경우 : 특정 판매월별을 기준으로 먼저 확인 후, 조건에 맞는 데이터를 걸러냄

5. 따라서, 다음과 같이 파티션 테이블 생성을 할 수 있음
```sql
CREATE TABLE SALES (
  PROD_ID  Number (6,0) NOT NULL,
  CUST_ID  Number (6,0) NOT NULL,
  CHANNEL_ID  Number (6, 0) NOT NULL,
  EMPLOYEE_ID Number (6, 0) NOT NULL,
  SALES_DATE Date DEFAULT SYSDATE NOT NULL,
  SALES_MONTH Varchar2(6 ),
  QUANTITY_SOLD Number (10,2),
  AMOUNT_SOLD Number (10,2),
  CREATE_DATE Date DEFAULT SYSDATE,
  UPDATE_DATE Date DEFAULT SYSDATE
)

PARTITION BY RANGE (SALES_MONTH) (
...

  PARTITION SALES_01_1998 VALUES LESS THAN ('199804') TABLESPACE MYTS,
  PARTITION SALES_02_1998 VALUES LESS THAN ('199807') TABLESPACE MYTS,
  PARTITION SALES_03-1998 VALUES LESS THAN ('199810') TABLESPACE MYTS,
  PARTITION SALES_04_1998 VALUES LESS THAN ('199901') TABLESPACE MYTS,
  ...
  PARTITION SALES_04_2003 VALUES LESS THAN ('200401") TABLESPACE MYTS
);
```
6. 이렇게 테이블을 생성한 뒤 INSERT를 하더라도 들어오는 데이터 값에 따라 자동으로 파티션별로 적재
7. 파티션 종류 : RANGE 파티션 / LIST 파티션 / 해시파티션 / 복합 파티션 존재
