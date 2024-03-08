-----
### JOIN
-----
1. 두 개 이상 테이블에 있는 컬럼의 값을 한 번에 가져오기 위해 사용하는 것
2. 형식
```sql
SELECT COL_NAME
FROM TABLE_NAME1, TABLE_NAME2;
```

3. 위처럼, 두 개 이상 테이블을 조인하게 되면, 다 대 다의 관계로 가져오게 됨
   - 테이블 1의 ROW의 수 X 테이블 2의 ROW의 수만큼 로우를 가져오게 됨
   - 즉, 테이블 1에 대해 테이블 2의 모든 행과 결합하게 되므로 위와 같은 공식 발생
    
4. EMP 테이블(14개 행)과 DEPT 테이블(4개 행) 두 개를 JOIN : 14 * 4 = 56행
```sql
SELECT *
FROM EMP, DEPT;
```

-----
### JOIN 구분
-----
1. 조인 연산자에 따른 구분 : EQUI-JOIN, ANTI-JOIN
2. 조인 대상에 따른 구분 : SELF-JOIN
3. 조인 조건에 따른 구분 : INNER-JOIN, OUTER-JOIN, SEMI-JOIN, CATASSIAN-JOIN
4. 기타 : ANSI-JOIN
   
-----
### JOIN 조건
-----
1. 두 개 이상의 테이블에서 가져온 결과 중 정확한 결과면, 가져오기 위해 공통 부분을 이용한 조건문 반드시 필요
2. 조건 절에는 두 테이블의 공통 부분에 대해 조건을 달아줘야함
3. 공통 컬럼 부분에 대한 조건을 하므로, 두 테이블에 대해 별칭을 달거나, 테이블 명을 명시시켜야함
4. JOIN은 공통 부분이 범위여도 가능
5. 형식
```sql
SELECT COL_NAME
FROM TABLE_NAME1, TABLE_NMAE
WHERE TABLE_NAME1의 공통 요소 = TABLE_NAME2의 공통 요소;
```

6. 위 조인에서 올바른 조인 방법
```sql
SELECT *
FROM EMP, DEPT
WHERE EMP.DEPTNO = DEPT.DEPTNO;
```

```sql
SELECT *
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DEPTNO;
```

7. 조인 조건을 컬럼 단위로 기술
   - A와 B 테이블에 대해 두 테이블이 공통적으로 가진 컬럼을 연결
   - 조인 결과가 참(True)에 해당하는, 두 컬럼의 값이 같은 행을 추출
     
-----
### 동등 조인(EQUI-JOIN)
-----
1. 가장 기본이 되며, 일반적인 조인 방법
2. WHERE절에 등호('=') 연산자를 이용해 2개 이상의 테이블이나 뷰를 연결한 조인
3. 등호 연산자를 사용한 WHERE절 조건(조인 조건)에 만족하는 데이터를 추출 
```sql
SELECT E.EMPNO, E.ENAME, D.DEPTNO, D.DNAME
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DEPTNO;
```

-----
### 예제
-----
1. 사원의 사원번호, 이름, 근무부서 이름을 가져온다.
```sql
SELECT E.EMPNO, E.ENAME, D.DNAME
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DEPTNO;
```

2. 사원의 사원번호, 이름, 근무지역을 가져온다.
```sql
SELECT E.EMPNO, E.ENAME, D.LOC
FROM EMP E, DEPT D
WHERE E.DEPTNO = D.DEPTNO;
```

3. DALLAS에 근무하고 있는 사원들의 사원번호, 이름, 직무를 가져온다.
```sql
SELECT E.EMPNO, E.ENAME, E.JOB
FROM EMP E, DEPT D
WHERE (E.DEPTNO = D.DEPTNO) AND (D.LOC = 'DALLAS');
```

4. SALES 부서에 근무하고 있는 사원들의 급여 평균을 가져온다.
```sql
SELECT AVG(E.SAL)
FROM EMP E, DEPT D
WHERE (E.DEPTNO = D.DEPTNO) AND (D.DNAME = 'SALES');
```

5. 1982년에 입사한 사원들의 사원번호, 이름, 입사일, 근무부서 이름을 가져온다.
```sql
SELECT E.ENMPNO, E.EMPNO, E.ENAME, E.HIREDATE, D.DNAME
FROM EMP E, DEPT D
WHERE (E.DEPTNO = D.DEPTNO) AND (E.HIREDATE BETWEEN '1982/01/01' AND '1982/12/31');
```

6. 각 사원들의 사원번호, 이름, 급여, 급여 등급을 가져온다. (조인은 범위의 조건도 가능)
```sql
SELECT E.EMPNO, E.ENAME, E.SAL, S.GRADE, 
FROM EMP E, SALGRADE S;
WHERE (E.SAL BETWEEN S.LOSAL AND S.HISAL);
```

7. SALES 부서에 근무하고 있는 사원의 사원번호, 이름, 급여등급을 가져온다.
```sql
SELECT E.EMPNO, E.ENAME, S.GRADE
FROM EMP E, DEPT D, SALGRADE S
WHERE (E.DEPTNO = D.DEPTNO) AND (E.SAL BETWEEN S.LOSAL AND S.HISAL) AND (D.DNAME = 'SALES');
```

8. 각 급여 등급별 총합과 평균, 사원의 수, 최대급여, 최소급여를 가져온다.
```sql
SELECT SUM(E.SAL), AVG(E.SAL), COUNT(E.EMPNO), MAX(E.SAL), MIN(E.SAL)
FROM EMP E, SALGRADE S
WHERE (E.SAL BETWEEN S.LOSAL AND S.HISAL) 
GROUP BY S.GRADE;
```

9. 급여 등급이 4등급인 사원들의 사원번호, 이름, 급여, 근무부서이름, 근무지역을 가져온다.
```sql
SELECT E.EMPNO, E.ENAME, E.SAL, D.DNAME, D.LOC
FROM EMP E, SALGRADE S, DEPT D
WHERE (E.DEPTNO = D.DEPTNO) AND (E.SAL BETWEEN S.LOSAL AND S.HISAL) AND (S.GRADE = 4);
```
