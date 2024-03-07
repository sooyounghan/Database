-----
### 추가적인 함수에 대한 공부는 지속적 필요
-----

-----
### Numeric Function
-----
: 컬럼에 저장되어 있는 숫자 값에 대해 처리하여 반환할 수 있는 함수를 의미

-----
### DUAL TABLE
-----
1. 특정 컬럼으로부터 데이터를 가져오는 것이 아니므로, 테이블 이름을 작성할 필요가 없음
2. DB에서는 SELECT 다음에 FROM문 작성해야 하는 것이 기본 문법
3. DUAL은 가상의 테이블로서, 테이블의 역할을 도와주는 테이블

```sql
SELECT *
FROM DUAL;
```
: 결과는 DUMMY 행의 값은 X

4. 10 + 10의 연산 결과를 확인하고 싶을 때는,
```sql
SELECT 10 + 10
FROM DUAL;
```
: 결과는 10 + 10행이며, 값은 20

-----
### 절댓값 함수 : ABS
-----
1. -10의 절댓값을 구하면,
```sql
SELECT -10, ABS(-10) FROM DUAL;
```
: 결과는 -10행 : -10 / ABS(-10) 행 : 10

2. 전 직원의 급여를 2000 삭감하고, 삭감한 급여액의 절댓값을 구한다.
```sql
UPDATE EMP01
SET SAL = SAL-2000;

SELECT ABS(SAL)
FROM EMP01;
```

```sql
SELECT ABS(SAL-2000)
FROM EMP01;
```

-----
### 소수자 이하 버림 : FLOOR
-----
1. 12.34 출력
```sql
SELECT 12.34 FROM DUAL;
```
: 값은 12.34

2. 12.34의 소수점 버림
```sql
SELECT FLOOR(12.34) FROM DUAL;
```
: 12

3. 급여가 1500 이상인 사원의 급여를 15% 삭감한다. 단, 소수점 이하는 버린다.
```sql
UPDATE EMP01
SET SAL = SAL * 0.85;
WHERE SAL >= 1500;

SELECT FLOOR(SAL)
FROM EMP01;
```

```sql
SELECT FLOOR(SAL * 0.85)
WHERE EMP01
WHERE SAL >= 1500;
```
-----
### 반올림 : ROUND (기본값 : 소수점 첫쨰 자리에서 반올림)
-----
1. 12.3456 출력
```sql
SELECT 12.3456
FROM DUAL;
```

2. 12.3456 반올림 출력
```
SELECT ROUND(12.3456)
FROM DUAL;
```

3. 12.8888 반올림 출력
```
SELECT ROUND(12.8888)
FROM DUAL;
```

4. 자릿수 조정 가능 : ROUND(
