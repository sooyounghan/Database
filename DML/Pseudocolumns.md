-----
### 의사컬럼 (Pseudo-column)
-----
1. 테이블의 컬럼처럼 동작하지만, 실제로 테이블에 저장되지 않는 컬럼
2. NEXTVAL, CURRVAL : Sequence에서 사용
3. ROWNUM
   - 쿼리에서 반환되는 각 로우들에 대한 순서 값을 나타내는 의사컬럼
   - ROWNUM은 최상위에서 N개만 추출 가능, 중간에 있는 N개를 추출 불가
4. ROWID
   - 각 행을 구분하기 위한 유일한 값 (데이터 파일, 데이터블럭, 블럭 내 위치 등)
   - 테이블에 저장된 각 ROW가 저장된 주소 값을 가리키는 의사컬럼 (각 ROW를 식별하는 값으로 유일한 값을 가짐)
   - B-Tree 인덱스는 테이블에서 인덱스 키로 잡혀있는 컬럼과 해당 테이블 로우를 찾아가기 위한 주소 정보 저장하는 것이 ROWID
```sql
SELECT ROWNUM, ROWID, EMPNO
FROM EMP;
```
-----
### ROWNUM
-----
1. 사원 테이블에서 사원정보, 사원 이름, 입사일에 대해 출력한다. (ROWNUM, ROWID)
```sql
SELECT ROWNUM, ROWID, EMPNO, ENAME, HIREDATE
FROM EMP;
```

2. 입사일이 가장 빠른 사원 3명을 조회하시오.
```sql
SELECT ROWNUM, A.ROWNUM_A, A.EMPNO, A.ENAME, A.HIREDATE
FROM (SELECT ROWNUM AS "ROWNUM_A", EMPNO, ENAME, HIREDATE
      FROM EMP
      ORDER BY HIREDATE) A
WHERE ROWNUM <= 3;
```

3. 이름순으로 앞에서 3명의 정보를 조회한다.
```sql
SELECT ROWNUM, A.ROWNUM_A, A.EMPNO, A.ENAME, A.HIREDATE
FROM (SELECT [ROWNUM AS "ROWNUM_A",] EMPNO, ENAME, HIREDATE
         FROM EMP
         ORDER BY ENAME) A
WHERE ROWNUM <= 3;
```

4. 이름순으로 앞에서 4~6번째까지 3명의 정보를 조회한다.
```sql
SELECT ROWNUM, [A.ROWNUM_A,] A.EMPNO, A.ENAME, A.HIREDATE
FROM (SELECT [ROWNUM AS "ROWNUM_A"], EMPNO, ENAME, HIREDATE
         FROM EMP
         ORDER BY ENAME) A
WHERE ROWNUM >= 4 AND ROWNUM <= 6; 
```
   - 에러 발생
   - 해결 방법 : 서브쿼리를 다시 한 번 중첩해 중첩된 Query문의 ROWNUM을 조건으로 사용

```sql
SELECT ROWNUM, [B.ROWNUM_A,] B.EMPNO, B.ENAME, B.HIREDATE
FROM (SELECT [ROWNUM AS "ROWNUM_A",] EMPNO, ENAME, HIREDATE
      FROM (SELECT EMPNO, ENAME, HIREDATE
            FROM EMP
            ORDER BY ENAME) A
     ) B
WHERE B.ROWNUM_A >= 4 AND B.ROWNUM_A <=6; 
```
