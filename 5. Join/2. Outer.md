-----
### Outer Join
-----
1. 조인 조건에 해당하지 않기 때문에 결과에 포함되지 않는 행(Row)까지 가져오는 조인 : 정보가 없는 부분에 대해 OUTER JOIN
2. 즉, 조인 조건에 만족하는 데이터 뿐만 아니라, 어느 한 쪽 테이블에 조인 조건에 명시된 컬럼 갑이 없거나(NULL 이더라도) 해당 로우가 아예 없는 경우라도 추출
3. 조인 조건에서 데이터가 없는 테이블의 컬럼에 (+) 기호를 붙여야 함

4. 각 사원의 이름, 사원번호, 직장 상사의 이름을 가져온다. 단, 직속 상관이 없는 사원도 가져온다.
   - E1 : 각 사원의 정보
   - E2 : 직장 상사의 정보 (직장 상사 중 KING은 직장 상사 번호가 없으므로 NULL 이므로 이 정보 포함해야함) : 없는 부분에 OUTER JOIN 표시
```sql
SELECT E1.ENAME, E1.EMPNO, E2.ENAME
FROM EMP E1, EMP E2
WHERE E1.MGR = E2.EMPNO(+);
```

5. 모든 부서의 소속 사원의 근무부서명, 사원번호, 사원이름, 급여를 가져온다.
   - D : 부서의 정보
   - E : 사원의 정보 (사원의 근무부서 번호 중 40번은 없음) : 없는 부분에 OUTER JOIN 표시
```sql
SELECT D.DNAME, E.EMPNO, E.ENAME, E.SAL
FROM DEPT D, EMP E
WHERE D.DEPTNO = E.DEPTNO(+);
```

-----
### Outer Join
-----
<div align = "center">
<img src = "https://github.com/sooyounghan/DataBase/assets/34672301/f7de2f0e-c542-49fe-8110-a8a6e63ff221">
</div>

      DEPARTMENT : 부서 정보가 담긴 테이블
      JOB_HISTORY : 직업 정보가 담긴 테이블
      
1. 일반적인 조인의 경우 (INNER-JOIN)
```sql
SELECT D.DEPARTMENT_ID, D.DEPARTMENT_NAME, J.JOB_ID, J.DEPARTMENT_ID
FROM DEPARTMENT D, JOB_HISTORY J
WHERE D.DEPARTMENT_ID = J.DEPARTMENT.ID;
```
<div align = "center>
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/575c3a09-2666-4b7d-bb66-0567bb521863">
</div>

   - JOB_HISTORY에 없는 부서는 조회되지 않음

2. 외부 조인 사용 (OUTER-JOIN)
```sql
SELECT D.DEPARTMENT_ID, D.DEPARTMENT_NAME, J.JOB_ID, J.DEPARTMENT_ID
FROM DEPARTMENT D, JOB_HISTORY J
WHERE D.DEPARTMENT_ID = J.DEPARTMENT_ID(+);
```
<div align = "center>
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/cc4d5ca0-f274-4da8-a9cd-4aa0b1873a4c">
</div>

   - 조인 조건에서 데이터가 없는 테이블의 컬럼에 (+)를 붙이는 것

3. 조인 조건에 모두 (+)를 붙여줘야 함
   - 모두 부여하지 않을 시 다음과 같이 일부 값만 출력
```sql
SELECT E.DEPARTMENT_ID, E.EMP_NAME, J.JOB_ID, J.DEPARTMENT_ID
FROM EMPLOYEES E, JOB_HISTORY J
WHERE (E.DEPARTMENT_ID = J.DEPARTMENT_ID) AND (E.EMPLOYEE_ID = J.EMPLOY_ID(+));
```
<div align = "center>
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/7b232297-313c-4430-bdb3-4d1d2e5a8c1e">
</div>

   - 조인 조건에 모두 부여할 시
```sql
SELECT E.DEPARTMENT_ID, E.EMP_NAME, J.JOB_ID, J.DEPARTMENT_ID
FROM EMPLOYEES E, JOB_HISTORY J
WHERE (E.DEPARTMENT_ID = J.DEPARTMENT_ID(+)) AND (E.EMPLOYEE_ID = J.EMPLOY_ID(+));
```
<div align = "center>
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/a7e596e7-7ef6-4357-a046-13d1e4f825c4">
</div>

4. OUTER-JOIN 주의사항
   - 조인 대상 테이블 중 데이터가 없는 테이블 조인 조건에 (+)를 붙인다.
   - 외부 조인의 조인 조건이 여러 개 일 때는, 모든 조건에 (+)를 붙인다.
   - 한 번에 한 테이블에만 외부 조인을 할 수 있음 (A, B, C 3개가 조인 대상 테이블인데, A-B가 외부 조인 관계면, B-C는 불가)
   - (+) 연산자가 붙은 조건과 OR를 같이 사용할 수 없음
   - (+) 연산자가 붙은 조건에는 IN 연산자를 같이 사용할 수 없음 (단, IN 절에 포함되는 값이 1개인 때는 가능)
  
-----
### CARTASIAN PRODUCT JOIN (CROSS JOIN)
-----
<div align = "center>
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/1114d0e7-c80f-4ae4-94b9-37ce3b1a1aea">
</div>

1. WHERE 절에 조인 조건이 없는 조인
2. FROM 절에 테이블을 명시했으나, 두 테이블 간 조인 조건이 없는 조인
3. 엄밀하게 말하면, FROM 절에 2개 이상 테이블을 명시했으므로 조인에 해당하며, 그 결과는 두 테이블의 건수의 곱
4. A 테이블의 건수가 n1, B 테이블의 건수가 n2이면, 결과는 n1 * n2
```sql
SELECT E.EMPLYOEE_ID, E.EMP_NAME, D.DEPARTMENT_ID, D.DEPARTMENT_NAME
FROM EMPLOYEES E, DEPARTMENT D;
```

<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/f4069edb-140d-4935-9158-fd1f59ad5651">
</div>
