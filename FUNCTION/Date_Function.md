-----
### 날짜 함수
-----
1. 날짜 데이터를 제어할 수 있는 함수들을 제공
   - ORACLE : 년/월/일 시:분:초
   - MySQL : 년/월/일
   - TIMESTAMP : 년/월/일 시:분:초 밀리초
2. SYSDATE : 현재 날짜와 시간을 반환
3. MONTHS_BETWEEN : 두 날짜간의 개월 수를 구함
4. ADD_MONTHS : 주어진 개월 수 만큼 더함

-----
### SYSDATE, SYSTIMESTMAP : 현재일자와 시간을 DATE와 TIMESTAMP형으로 반환
------
```sql
SELECT SYSDATE, SYSTIMESTMAP
FROM DUAL;
```
: 현재 날짜와 시간, 현재 날짜와 시간 출력

1. SYSDATE, SYSTIMESTAMP 연산
```sql
SELECT SYSDATE - 10000
FROM DUAL;
```
: 현재 시점을 기준으로 10000일 전의 시간 출력 (연산은 일의 단위)

2. 각 사원의 입사한 날짜로부터 1000일 후가 되는 날짜를 가져온다.
```sql
SELECT HIREDATE + 1000
FROM EMP;
```

3. 직무가 'SALESMAN'인 사원의 입사일 100일 전 날짜를 구한다.
```sql
SELECT HIREDATE - 100
FROM EMP
WHERE JOB = 'SALESMAN';
```

4. MySQL
   - CURDATE(): "YYYY-MM-DD"형식, 날짜만 출력 (예)2024-03-06)
   - CURRENT_DATE() : CURDATE()와 동일
   - NOW() : "YYYY-MM-DD HH24 : MM : SS"형식, 날짜와 시간 출력 (예)2024-03-06 14 : 06 : 25)

-----
### ADD_MONTHS(date, Integer) : 매개변수로 들어온 날짜에 Integer 만큼의 월을 더한 날짜 반환
------
```sql
SELECT ADD_MONTHS(SYSDATE, 1), ADD_MONTHS(SYSDATE, -1)
FROM DUAL;
```
: 현재 날짜에서 1개월 추가, 현재 날짜에서 1개월 감소

- 각 사원들의 입사일 후 100개월 되는 날짜를 구한다.
```sql
SELECT ADD_MONTHS(HIRDATE, 100)
FROM EMP;
```

-----
### MONTHS_BETWEEN(date1, date2) : 두 날짜 사이의 개월 수 반환
------
1. date1 < date2 : 음수
2. date1 > date2 : 양수

```sql
SELECT MONTHS_BETWEEN(SYSDATE, ADD_MONTHS(SYSDATE, 1)), MONTHS_BETWEEN(ADD_MONTHS(SYSDATE, 1), SYSDATE)
FROM DUAL;
```
: -1, 1

```sql
SELECT TRUNC(MONTHS_BETWEEN(SYSDATE, HIRDATE))
FROM DUAL;
```

-----
### LAST_DAY(date) : date 날짜 기준 해당 월의 마지막 일자 반환
------
```sql
SELECT LAST_DAY(SYSDATE)
FROM DUAL;
```
  - 예시(현재 시간 : 2015-03-01) : 2015-03-31 현재시간

-----
### ROUND(date, format), TRUNC(date, format) : 날짜 함수에도 적용 가능
------
1. ROUND : format 다음 자리에 따라 반올림한 날짜
2. TRUNC : format에 잘라낸 날짜

```sql
SELECT SYSDATE, ROUND(SYSDATE, 'MONTH'), TRUNC(SYSDATE, 'MONTH')
FROM DUAL;
```
  - 예시(현재 시간 : 2015-03-16) : 2015-03-16, 2015-04-01, 2015-03-01 현재시간
  - ROUND 함수 ; 현재가 16일이므로 월(MONTH)를 기준으로 반올림하면, 다음달로 넘어감
  - TRUNC 함수 : 월을 기준으로 무조건 잘라냄

```sql
SELECT ROUND(SYSDATE, 'CC') AS "년도 두자리", ROUND(SYSDATE, 'YYYY') AS "월 기준",
       ROUND(SYSDATE, 'MM') AS "일 기준", ROUND(SYSDATE, 'DAY') AS "주 기준",
       ROUND(SYSDATE, 'DDD') AS "시 기준", ROUND(SYSDATE, 'HH') AS "분 기준"
       ROUND(SYSDATE, 'MI') AS "초 기준"     
FROM DUAL;
```

    01/01/01, 24/01/01, 
    24/03/01, 24/03/10(일주일 기준 절반 이전 : 일요일, 이후 : 토요일),
    24/03/09(오후 시간이면 반올림), 24/03/08(타입 : 년/월/일이라 시분초가 보이지 않음)
    24/03/08(타입 : 년/월/일이라 시분초가 보이지 않음)

```sql
SELECT TRUNC(SYSDATE, 'CC') AS "년도 두자리", TRUNC(SYSDATE, 'YYYY') AS "월 기준",
       TRUNC(SYSDATE, 'MM') AS "일 기준", TRUNC(SYSDATE, 'DAY') AS "주 기준",
       TRUNC(SYSDATE, 'DDD') AS "시 기준", TRUNC(SYSDATE, 'HH') AS "분 기준"
       TRUNC(SYSDATE, 'MI') AS "초 기준"     
FROM DUAL;
```

    01/01/01, 24/01/01, 
    24/03/01, 24/03/03
    24/03/08, 24/03/08(타입 : 년/월/일이라 시분초가 보이지 않음)
    24/03/08(타입 : 년/월/일이라 시분초가 보이지 않음)
    
3. 전 사원의 근무일을 가져온다.
```sql
SELECT TRUNC(SYSDATE - HIREDATE)
FROM EMP;
```

4. 각 사원의 입사일을 월 기준으로 반올림한다.
```sql
SELECT ROUND(HIREDATE, 'YYYY')
FROM EMP;
```

5. 1981년에 입사한 사원들의 사원번호, 사원이름, 급여, 입사일을 가져온다.
```sql
SELECT EMPNO, ENAME, SAL, HIREDATE
FROM EMP
WHERE HIREDATE BETWEEN '1981/01/01' AND '1981/12/31';
```

```sql
SELECT EMPNO, ENAME, HIREDATE
FROM EMP
WHERE TRUNC(HIREDATE, 'YYYY') = '1981/01/01';
```

-----
### NEXT_DAY(date, char) : date를 char에 명시한 날짜로 다음 주 중 일자로 반환
----- 
```sql
SELECT NEXT_DAY(SYSDATE, '금요일')
FROM DUAL;
```
: 2024-03-15 현재시간 
