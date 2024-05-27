-----
### DELETE
-----
1. 테이블 내 행(ROW)를 제거하는 구문
```sql
DELETE FROM TABLE_NAME
[WHERE 조건절];
```

2. 다음과 같이 EMP01 테이블이 있다고 하자.
```sql
CREATE TABLE EMP01
AS
SELECT * FROM EMP;
```

3. EMP01 내 모든 행을 제거한다.
```sql
DELETE FROM EMP01;
```

4. 사원번호가 7499번인 사원의 정보를 삭제한다.
```sql
DELETE FROM EMP01 WHERE EMPNO = 7499;
```

5. 사원의 급여가 평균 급여 이하인 사원의 정보를 삭제한다.
```sql
DELETE FROM EMP01
WHERE SAL <= (SELECT AVG(SAL) FROM EMP01);
```

6. 커미션을 받지 않는 사원의 정보를 삭제한다.
```sql
DELETE FROM EMP01 WHERE COMM IS NULL;
```
