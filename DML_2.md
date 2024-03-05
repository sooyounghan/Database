-----
### Like 연산자
-----
1. 조건문에서 문자열 컬럼도 = 와 <>로 비교가 가능
2. 만약 문자열 컬럼에 저장되어 있는 값이 특정 문자열을 포함하고 있는지 파악하고 싶으면 Like 연산자 사용

```sql
SELECT COL_NAME
FROM TABLE_NAME
WHERE COL_NAME LIKE '와일드카드';
```

3. 와일드카드
   - _ : 글자 하나를 의미
   - % : 글자 0개 이상을 의미
   - 위 두개의 조합으로 이루어짐


         예) _A_ : 세 글자 중 두 번째 글자만 A인 문자
         예) %A% : 글자 수의 제한은 없으나 중간에 A 문자 포함

4. 이름이 F로 시작하는 사원의 이름과 사원 번호를 가져온다.
```sql
SELECT ENAME, EMPNO
FROM EMP
WHERE ENAME LIKE 'F%';
```

5. 이름이 S로 끝나는 사원의 이름과 사원 번호를 가져온다.
```sql
SELECT ENAME, EMPNO
FROM EMP
WHERE ENAME LIKE '%S';
```

6. 이름에 A가 포함되어 있는 사원의 이름과 사원 번호를 가져온다.
```sql
SELECT ENAME, EMPNO
FROM EMP
WHERE ENAME LIKE '%A%';
```

7. 이름의 두 번째 글자가 A인 사원의 사원 이름, 사원 번호를 가져온다.
```sql
SELECT ENAME, EMPNO
FROM EMP
WHERE ENAME LIKE '_A%';
```

8. 이름이 4글자인 사원의 사원 이름, 사원 번호를 가져온다.
```sql
SELECT ENAME, EMPNO
SELECT EMP
WHERE ENAME LIKE '____';
```

-----
### NULL
-----
1. Oracle에서 NULL : 정해져 있지 않은 값 또는 무한대의 값을 의미
2. = 이나 <>를 통해 컬럼의 값이 NULL 인지 연산할 수 없음
3. IS NULL 또는 IS NOT NULL을 통해 NULL 비교

4. 사원 중에 커미션을 받지 않는 사원의 사원번호, 이름, 커미션을 가져온다.
```sql
SELECT EMPNO, ENAME, COMM
FROM EMP
WHERE COMM IS NULL;
```

5. 회사 대표(직속상관이 없는 사람)의 이름과 사원번호를 가져온다.
```sql
SELECT ENAME, EMPNO
FROM EMP
WHERE MGR IS NULL;
```
