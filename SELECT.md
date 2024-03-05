-----
### SELECT문
-----
1. 데이터베이스에 저장된 데이터를 가져오는 명령문
2. 상황이나 조건에 맞는 데이터를 추출 가능
3. 기본 구조

```sql
SELECT {* | [테이블명.]컬럼명 1, .. 컬럼명 N}
FROM [데이터베이스명.]테이블명
[WHERE 조건]
[GROUP BY 그룹기준]
[HAVING 그룹조건]
[ORDER BY 정렬 기준 정렬 방법];
```

<div align = "center">
<img src="https://github.com/sooyounghan/Data-Structure/assets/34672301/7ce05962-a93d-4379-a93d-fb60058f5bf5">
</div>

-----
### SELECT - 모든 Column의 데이터 가져오기
-----
* 항상 키워드 - FROM절 - 나머지 적기
  
```sql
SELECT *
FROM TABLE_NAME;
```
1. < DEPT > TABLE의 모든 정보를 가져옴
```sql
SELECT *
FROM DEPT;
```

2. < EMP > TABLE의 모든 정보를 가져옴
```sql
SELECT *
FROM EMP;
```

-----
### SELECT - 특정 Column의 데이터 가져오기
-----
```sql
SELECT COL1, COL2(COL1과 COL2는 TABLE에 있는 COLUMN)
FROM TABLE_NAME;
```

1. < EMP > TABLE에서 사원의 이름과 사원 번호를 가져옴
```sql
SELECT ENAME, EMPNO
FROM EMP;
```

2. < EMP > TABLE에서 사원의 이름과 사원 번호, 직무, 급여를 가져옴
```sql
SELECT ENAME, EMPNO, JOB, SAL
FROM EMP;
```

3. < DEPT > TABLE에서 부서 번호와 부서 이름을 가져옴
```sql
SELECT DEPNO, DANME
FROM DEPT;
```

