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
### ABS(n) : 절댓값 반환 함수
-----
1. -10의 절댓값을 구하면,
```sql
SELECT -10, ABS(-10) FROM DUAL;
```
: 결과는 -10행 : -10 / ABS(-10) 행 : 10

2. 전 직원의 급여를 2000 삭감하고, 삭감한 급여액의 절댓값을 구한다.
```sql
UPDATE EMP01
SET SAL = ABS(SAL-2000);
```

```sql
SELECT ABS(SAL-2000)
FROM EMP01;
```

-----
### FLOOR(n) : 매개변수 n보다 작거나 가장 큰 정수 반환
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
SET SAL = FLOOR(SAL * 0.85);
WHERE SAL >= 1500;
```

```sql
SELECT FLOOR(SAL * 0.85)
WHERE EMP01
WHERE SAL >= 1500;
```

-----
### CEIL(n) : 매개변수 n과 같거나 가장 큰 정수 반환
-----
```sql
SELECT CEIL(10.123), CEIL(10.541), CEIL(11.001)
FROM DUAL;
```
: 11, 11, 12

-----
### ROUND(n, i) : 매개변수 n을 소수점 (i + 1)의 자리에서 반올림 (i는 생략 가능) [디폴트 값 : 0]
-----
1. 12.3456 출력
```sql
SELECT 12.3456
FROM DUAL;
```

2. ROUND(n) : 매개변수 n을 소수점 1번째 자리에서 반올림 [디폴트값]
   - 12.3456 반올림 출력
```sql
SELECT ROUND(12.3456)
FROM DUAL;
```

3. 12.8888 반올림 출력
```sql
SELECT ROUND(12.8888)
FROM DUAL;
```

4. ROUND(n, i) : 매개변수 n을 소수점 (i + 1)의 자리에서 반올림 (i는 생략 가능)
```sql
SELECT ROUND(12.8888, 2)
FROM DUAL;
```
: 값은 12.9

5. 음수도 가능 : 소수점을 기준으로 왼쪽 i번쨰에서 반올림 발생
```sql
SELECT ROUND(1222.8888, -2)
FROM DUAL;
```
: 십의 자리에서 반올림 (1200)

6. 급여가 2000 이하인 사원들의 급여를 20%씩 인상한다. 단, 10의 자리를 기준으로 반올림한다.
```sql
UPDATE EMP
SET SAL = ROUND(SAL * 1.2, -2)
WHERE SAL <= 2000;
```
```sql
SELECT ROUND(SAL * 1.2, -2)
FROM EMP
WHERE SAL <= 2000;
```
-----
### TRUNCATE(TRUNC(n1, n2)) : n1을 소수점 n2자리에서 무조건 버림 반환 (기본값 : n2 = 0)
-----
```sql
SELECT TRUNC(1112.3456) FROM DUAL;
```

1. TRUNC(n1, n2) : 소수점 지정 가능(양수일 때는 소수점 오른쪽 자리 버림)
```sql
SELECT TRUNC(1112.3456, 2) FROM DUAL;
```
: 1112.34

2. TRUNC(n1, n2) : 음수도 가능(음수일 때는 소수점 왼쪽 자리 버림)
```sql
SELECT TRUNC(1112.3456, -2) FROM DUAL;
```
: 1100

3. 전 직원의 급여를 십의 자리 이하를 삭감한다.
```sql
UPDATE EMP
SET SAL = TRUNC(SAL, -2);
```
```sql
SELECT TRUNC(SAL, -2)
FROM EMP;
```

-----
### MOD(n1, n2) : n1을 n2로 나눈 나머지를 구하는 연산자
-----
: n2 - n1 * FLOOR(n2 / n1)
```sql
SELECT MOD(10, 3), MOD(10, 4)
FROM DUAL;
```
: 결과값 : 3, 2

```sql
SELECT MOD(19, 4)
FROM DUAL;
```
: 결과값 : 3

-----
### REMAINDER(n1, n2) : n1을 n2로 나눈 나머지 값 구하는 연산자
-----
: n2 - n1 * ROUND(n2 / n1)

```sql
SELECT REMAINDER(19, 4)
FROM DUAL;
```
: 결과값 : -1

-----
### POWER(n1, n2) : n1를 n2제곱 한 결과 반환 
-----
1. n1는 정수 / 실수 모두 가능
2. n2가 음수면 n1은 정수만 가능
```sql
SELECT POWER(3, 2), POWER(3, 3), POWER(3, 3.0001)
FROM DUAL;
```
: 9, 27, 27.0029664

-----
### SQRT(n) : n의 제곱근을 반환한다. 
-----
```sql
SELECT SQRT(2), SQRT(5)
FROM DUAL;
```
: 1.41421356, 2.23606798
