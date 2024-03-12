-----
### ANSI-JOIN
-----
1. ANSI SQL 문법을 사용한 조인이며, INNER-JOIN / OUTER-JOIN / CROSS-JOIN 모두 변환 가능
2. 다른 점은 WHERE 절에 조인 조건이 들어가는 것이 아닌 FROM 절에 조인 조건 포함

-----
### ANSI INNER JOIN
-----
1. 기존 INNER JOIN
```sql
SELECT A.컬럼1, A.컬럼2, B.컬럼1, B.컬럼2
FROM 테이블 A, 테이블 B
WHERE A.컬럼1 = B.컬럼1 (조인조건)
...;
```

2. ANSI 문법
```sql
SELECT A.컬럼1, A.컬럼2, B.컬럼1, B.컬럼2
FROM 테이블 A INNER JOIN 테이블 B ON (A.컬럼1 = B.컬럼1) [조인 조건]
WHERE ..;
```

  - FROM절에 INNER JOIN 구문 작성
  - 조인 조건은 ON절에 명시하고, 조인 조건 외에 조건은 기존대로 WHERE 절에 명시

3. 예제
   - INNER-JOIN
```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENT D
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID;
```

  - ANSI 문법
```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E INNER JPIN DEPARTMENT D ON (E.DEPARTMENT_ID = D.DEPARTMENT_ID);
```

4. 조인 조건 컬럼이 두 테이블 모두 동일하면 ON 대신, USING절 사용 가능하지만, 이 때 SELECT 및 ORDER BY 절과 같은 다른 절에 테이블명.컬럼명이 아닌 컬럼명 작성
```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E INNER JOIN DEPARTMENT D USING (E.DEPARTMENT_ID = D.DEPARTMENT_ID);
ORDER BY DEPARTMENT_ID;
```

-----
### NATURAL JOIN
-----
1. 테이블 간에 동일한 이름을 가진 열을 찾아서 자동으로 조인
2. 즉, 공통 열 이름을 찾아서 해당 열을 기준으로 조인
```sql
SELECT E.EMPNO, E.ENAME, DEPTNO, D.DNAME
FROM EMP E NATURAL JOIN DEPT D
```

3. 동일한 이름을 가진 열을 기준으로 조인하므로 결과 집합에서 중복된 열이 자동으로 제거
4. 즉, 결과 값은 해당 열이 일치하는 모든 행을 반환
5. 추가적으로, 조인의 대상이 되는 컬럼에 대해서는 한정자를 붙일 수 없음 (DEPTNO와 같이 한정자 제외 / 이 외 조인은 한정자가 필수)

-----
### ANSI OUTER JOIN
-----
1. 기존 문법
```sql
SELECT A.컬럼1, A.컬럼2, B.컬럼1, B.컬럼2
FROM 테이블 A, 테이블 B
WHERE A.컬럼1 = B.컬럼1(+) (조인조건)
...;
```
  - 기준 테이블과 대상 테이블(데이터가 없는 테이블)에서 대상 테이블 쪽 조인 조건에 (+) 부여
    
2. ANSI 문법
```sql
SELECT A.컬럼1, A.컬럼2, B.컬럼1, B.컬럼2
FROM 테이블 A LEFT(RIGHT) [OUTER] JOIN 테이블 B ON (A.컬럼1 = B.컬럼1) [조인 조건]
WHERE ..;
```
  - FROM 절에 명시된 테이블 순서에 입각해 먼저 명시된 테이블 기준으로 LEFT 혹은 RIGHT 붙임

3. 예제
   - OUTER-JOIN
```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENT D
WHERE (E.DEPARTMENT_ID = D.DEPARTMENT_ID(+)) AND (E.EMPLOYEES_ID = D.EMPLOYEES_ID(+));
```

  - ANSI 문법
```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E RIGHT OUTER JOIN DEPARTMENT D ON ((E.DEPARTMENT_ID = D.DEPARTMENT_ID) AND (E.EMPLOYEES_ID = D.EMPLOYEES_ID));
```

```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM DEPARTMENT D LEFT OUTER JOIN EMPLOYEES E ON ((E.DEPARTMENT_ID = D.DEPARTMENT_ID) AND (E.EMPLOYEES_ID = D.EMPLOYEES_ID));
```

-----
### ANSI CROSS JOIN
-----
1. 기존 문법
```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENT D;
```

2. ANSI 문법
```sql
SELECT E.EMPLOYEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E CROSS JOIN DEPARTMENT D;
```

-----
### FULL OUTER JOIN
-----
1. 외부 조인 중 하나로 ANSI 조인에서만 제공하는 기능
2. 두 테이블 모두 테이블의 일부 값이 없는 경우, 이 값을 모두 포함해서 출력하고 싶은 경우

3. 다음과 같이 테이블과 데이터를 생성 후 삽입했다고 하자.
<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/6b6e6b07-b552-4ca3-9180-bccf9db3cd87">
</div>

4. 외부 조인을 A 테이블 기준으로 하면 30이, B 테이블 기준으로 하면 40이 빠지는데, 이를 해결하는 방법은 다음과 같이 FULL OUTER JOIN
```sql
SELECT A.EMP_ID, B.EMP_ID
FROM HONG_A A FULL OUTER JOIN HONG_B B ON (A.EMP_ID = B.EMP_ID);
```

<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/7aaafb71-1263-4ed1-a303-8801e798a85c">
</div>
