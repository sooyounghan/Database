-----
### SEMI-JOIN
-----
1. 서브 쿼리를 이용해 서브 쿼리에 존재하는 데이터만 메인 쿼리에서 추출하는 조인 방법
2. IN과 EXIST 연산자를 사용한 조인
3. 서브 쿼리에 존재하는 메인 쿼리 데이터가 여러 건 존재하더라도 최종 반환되는 메인 쿼리 데이터는 중복되는 건이 없음
4. 서브 쿼리에 있는 테이블을 B, 메인 쿼리에 사용된 테이블을 A라고 한다면, B 테이블에 존재하는 A 테이블의 데이터를 추출하는 조인

5. EXISTS 사용
```sql
SELECT D.DEPTNO, D.DNAME
FROM DEPT D
WHERE EXIST (SELECT *
             FROM EMP E
             WHERE D.DEPTNO = E.DEPTNO
             AND E.SAL > 3000)
ORDER BY D.DNAME;
```

5. IN 사용
```sql
SELECT D.DEPTNO, D.DNAME
FROM DEPT D
WHERE D.DEPTNO IN (SELECT E.DEPTNO
                   FROM EMP E
                   WHERE E.SAL > 3000)
ORDER BY D.DNAME;
```

6. IN 연산자는 OR 조건으로 변환할 수 있고, EXISTS는 조건에 만족하는 데이터가 하나라도 있으면 결과를 즉시 반환

-----
### ANTI-JOIN
-----
1. 서브 쿼리의 B 테이블에는 없는 메인 쿼리의 A 테이블의 데이터만 추출하는 조인 방법
2. 즉, 한쪽 테이블에만 있는 데이터를 추출하는 것
3. 그러므로, NOT IN과 NOT EXISTS 연산자를 사용 (세미 조인과 반대)
4. NOT IN 연산자
```sql
SELECT E.EMPNO, E.ENAME, E.DEPTNO, D.DNAME
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DEPTNO
AND E.DEPTNO NOT IN (SELECT DEPTNO
                     FROM DEPT
                     WHERE MGR_ID IS NULL);
```
5. NOT EXIST 연산자
```sql
SELECT COUNT(*)
FROM EMP E
AND NOT EXIST (SELECT 1
               FROM DEPT D
               WHERE E.DEPTNO = D.DEPTNO
               AND MGR_ID IS NULL);
```
