-----
### 시퀀스 (Sequence)
-----
1. 테이블 내 컬럼 중 Primary Key를 지정하기 애매한 경우 1부터 1씩 증가되는 값을 저장하는 컬럼을 추가해 사용
2. 이 때, 1부터 1씩 증가되는 값을 구하기 위해 시퀀스를 사용
3. 즉, 자동 순번을 반환하는 데이터베이스 객체
4. 형식
```sql
CREATE SEQUENCE [스키마명.]시퀀스명
INCREMENT BY 증감숫자
START WITH 시작숫자
NOMINVALUE | MINVALUE 최솟값
NOMAXVALUE | MAXVLAUE 최댓값
NOCYCLE | CYCLE
NOCACHE | CACHE;
```
  - INCREMENT BY 증감숫자 : 증감숫자는 0이 아닌 정수, 양수이면 증가, 음수이면 감소(디폴트 : 1)
  - START WITH 시작숫자 : 시작 값으로, 절대 최소 값보다 작을 수 없음 (한 번 설정하면 변경 불가)
  - NOMINVALUE : 1
  - MINVALUE 최솟값 : 시퀀스가 가질 수 있는 최소값. 최솟값은 시작숫자와 작거나 같아야 하고, MAXVALUE보다 작아야함
  - NOMAXVALUE : 10의 27승
  - MAXVALUE 최댓값 : 시퀀스가 가질 수 있는 최댓값. 최댓값은 시작숫자와 같거나 커야하고, MINVALUE보다 커야함
  - NOCYCLE : 디폴트 값으로 최대나 최솟값에 도달하면 생성 중지
  - CYCLE : 최대 혹은 최솟값까지 갈 경우 순환
  - NOCACHE : 디폴트로 메모리에 시퀀스 값을 미리 할당해 놓지 않으며, 디폴트 값은 20
  - CACHE : 시퀀스를 메모리상에서 관리할 수 있도록 설정

5. 시퀀스 삭제
```sql
DROP SEQUENCE [스키마명.]시퀀스명;
```

6. 시퀀스 값 추출
   - SEQUENCE명.NEXTVAL : 다음 시퀀스 값을 가져옴
   - SEQUENCE명.CURRVAL : 현재 시퀀스 값을 가져옴
   - 현재 시퀀스 값을 가져오기 위해서는 반드시 다음 시퀀스 값을 먼저 가져와야 현재 시퀀스 값을 가져올 수 있음
   
-----
### 예제
-----
1. TEST_TABLE1 을 생성한다.
```sql
CREATE TABLE TEST_TABLE1 (
IDX NUMBER CONSTRAINT TEST_TABLE1_IDX_PK PRIMARY KEY,
NUMBER_DATA NUMBER NOT NULL
);
```

2. SEQUENCE를 생성한다.
```sql
CREATE SEQUENCE TEST_SEQ1
START WITH 0
INCREMENT BY 1
MINVALUE 0;
```

3. 현재 시퀀스의 값을 가져온다.
```sql
SELECT TEST_SEQ1.CURRVAL FROM DUAL;
```
: 에러 발생 (다음 시퀀스 값을 가져올 수 없음 (NEXTVAL 미사용)

4. TEST_TABLE1에 데이터를 삽입한다.
```sql
INSERT INTO TEST_TABLE1(IDX, NUMBER_DATA)
VALUES (TEST_SEQ1.NEXTVAL, 100);
```
```sql
INSERT INTO TEST_TABLE1(IDX, NUMBER_DATA)
VALUES (TEST_SEQ1.NEXTVAL, 200);
```
: (1, 100), (2, 200) 저장 (처음 초기 숫자 0이었으며, INSERT에서 NEXTVAL로 1씩 증가)

5. 다시 한 번, 현재 시퀀스의 값을 가져온다.
```sql
SELECT TEST_SEQ1.CURRVAL FROM DUAL;
```
: 2 조회 (현재 2까지 진행되었으므로, 2 출력)

6. TEST_SEQ1 시퀀스를 삭제한다.
```sql
DROP SEQUENCE TEST_SEQ1;
```

