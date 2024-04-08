-----
### NVL(expr1, expr2) / NVL2(expr1, expr2, expr3)
-----
1. NVL(expr1, expr2) : expr1이 NULL일 때, expr2 출력

2. NVL2(expr1, expr2, expr3) : expr1이 NULL이 아니면, expr2 출력, NULL이면 expr3 출력

-----
### COALESCE(expr1, expr2, expr3, ..., exprN) : 매개변수로 들어오는 표현식에서 NULL이 아닌 첫 번째 표현식 반환
-----
```sql
SELECT EMPNO, SAL, COMM COALESCE(SALARY * COMM, SALARY) AS SALARY;
FROM EMP;
```
1. SALARY * COMM의 결과값이 NULL이면 SALARY를, NULL이 아니면 SALARY * COMM 값을 반환
2. NULL과 수식연산자를 이용해 NULL과 연산하면 상대값이 무엇이든 NULL
3. expr1이 NULL이 아니면 expr1 값 반환, NULL이면 expr2 반환, expr2이 NULL이면 expr3..
   NULL이면 exprN 반환  
-----
### LNNVL(조건식) : 매개변수로 들어오는 조건식 결과가 FALSE이나 UNKNOWN이면 TRUE, TURE이면 FALSE
-----
1. 커미션이 0.2 이하인 사원 조회
```sql
SELECT EMPNO, COMM
FROM EMP
WHERE LNNVL(COMM >= 0.2);
```

2. LNNVL은 매개변수로 들어오는 조건의 결과가 TRUE이면 FALSE, FALSE나 UNKNOWN이면 TRUE 반환 (반대)

-----
### NULLIF(expr1. expr2) : expr1과 expr2를 비교해 같으면 NULL, 같지 않으면 expr1 반환
-----

-----
### 기타 함수
-----
-----
### GREATEST(expr1, expr2) : 매개변수로 들어오는 표현식 중 가장 큰 값
-----
```sql
SELECT GREATEST(1, 2, 3, 2)
FROM DUAL;
```
: 3

```sql
SELECT GREATEST('이순신', '강감찬', '세종대왕')
FROM DUAL;
```
: 이순신 (사전순)

-----
### LEAST(expr1, expr2) : 매개변수로 들어오는 표현식 중 가장 작은 값
-----
```sql
SELECT LEAST(1, 2, 3, 2)
FROM DUAL;
```
: 1

```sql
SELECT GREATEST('이순신', '강감찬', '세종대왕')
FROM DUAL;
```
: 강감찬 (사전순)

-----
### DECODE(expr1, search1, reusult1, search2, result2, ... default)
-----
1. expr1과 search1을 비교해 두 값이 같으면 result1, 같지 않으면 search2을 비교해 값이 같으면 reuslt2 반환
2. 최종적으로 같은 값이 없으면 default 반환

```sql
SELECT DECODE(EMPNO, 1, 'Direct', 2, 'Redirect', 3, ..., 'Others') DECODES
FROM EMP;
```
3. CASE 처리와 비슷
