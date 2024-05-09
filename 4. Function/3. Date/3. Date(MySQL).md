-----
### 날짜함수
-----
: 날짜에 대한 작업을 하는 함수

-----
### NOW(), SYSDATE(), CURRENT_TIMESTAMP()
-----
1. 현재 날짜와 시간 반환
2. 예제 : 현재 날짜와 시간을 가져옴
```sql
SELECT NOW(), SYSDATE(), CURRENT_TIMESTAMP()
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/cab2e063-94d9-4a50-bec3-900ce3325b0a">
</div>

-----
### CURDATE(), CURRENT_DATE()
-----
1. 현재 날짜를 반환
2. 예제 : 현재 날짜를 가져옴
```sql
SELECT CURDATE(), CURRENT_DATE()
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/4bfb2449-6555-4ace-96c7-7bdc254e7026">
</div>

-----
### CURTIME(), CURRENT_TIME()
-----
1. 현재 시간을 반환
2. 예제 : 현재 시간을 가져옴
```sql
SELECT CURTIME(), CURRENT_TIME()
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/41dc7f53-faaf-41c6-9348-e482e6513740">
</div>

-----
### DATE_ADD(날짜, INTERVAL 기준값 (단위 : YEAR, MONTH, DAY, HOUR, MINUTE, SECOND))
-----
1. 주어진 날짜에서 기준값 만큼 더함 
2. 예제 : 현재 날짜와 시간과 100일 후 날짜와 시간을 가져옴
```sql
SELECT NOW(), DATE_ADD(NOW(), INTERVAL 100 DAY)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/f21ac059-26e3-45f6-a0b7-cdba83c37df7">
</div>

3. 전체 사원들의 입사일과 입사일로부터 100일 후의 일자를 가져옴
```sql
SELECT HIRE_DATE, DATE_ADD(HIRE_DATE, INTERVAL 100 DAY)
FROM EMPLOYEES;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/7c9fe0eb-727c-41fe-b8d2-40992bda6b4f">
</div>

-----
### DATE_SUB(날짜, INTERVAL 기준값 (단위 : YEAR, MONTH, DAY, HOUR, MINUTE, SECOND))
-----
1. 주어진 날짜에서 기준값 만큼 뺌
2. 예제 : 현재 날짜와 시간과 100일 전 날짜와 시간을 가져옴
```sql
SELECT NOW(), DATE_SUB(NOW(), INTERVAL 100 DAY)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/cc085b58-9388-457b-91cc-da65ef95137c">
</div>

3. 전체 사원들의 입사일과 입사일로부터 100일 전의 일자를 가져옴
```sql
SELECT HIRE_DATE, DATE_SUB(HIRE_DATE, INTERVAL 100 DAY)
FROM EMPLOYEES;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/a74b099a-03e9-4b39-b2a5-f883fb253f10">
</div>

-----
### YEAR(날짜)
-----
1. 날짜의 연도를 가져옴
2. 예제
```sql
SELECT NOW(), YEAR(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/f25ee21d-c1b3-4c0a-b870-b9066be67d26">
</div>

-----
### MONTH(날짜)
-----
1. 날짜의 월을 가져옴
2. 예제
```sql
SELECT NOW(), MONTH(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/27375780-1fde-4a41-ba76-e8ddb34fd98d">
</div>

-----
### MONTHNAME(날짜)
-----
1. 날짜의 월을 영어로 가져옴
2. 예제
```sql
SELECT NOW(), MONTHNAME(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/f64d0fdf-3129-4f84-969f-74d325029e98">
</div>

-----
### DAYNAME(날짜)
-----
1. 날짜의 요일을 영어로 가져옴
2. 예제
```sql
SELECT NOW(), DAYNAME(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/c65f5a4a-2a09-4c34-b0d8-dce1db8be613">
</div>

-----
### DAYOFMONTH(날짜)
-----
1. 날짜의 월별 일자를 가져옴
2. 예제
```sql
SELECT NOW(), DAYOFMONTH(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/5b13d151-bb08-4011-afb8-0d37cd33953a">
</div>

-----
### DAYOFWEEK(날짜)
-----
1. 날짜의 주별 일자를 가져옴 (일요일 - 1, 월요일 - 2, 화요일 - 3, ...)
2. 예제
```sql
SELECT NOW(), DAYOFWEEK(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/7c441b58-d577-412e-9ba2-a748aa706ea0">
</div>

-----
### WEEKDAY(날짜)
-----
1. 날짜의 주별 일자를 가져옴 (월요일 - 0, 화요일 - 1, 수요일 - 2, ...)
2. 예제
```sql
SELECT NOW(), WEEKDAY(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/cd0f1e53-6323-4ac3-8944-ded20b679ac1">
</div>

-----
### DAYOFYEAR(날짜)
-----
1. 일년을 기준으로 한 날짜까지의 날 수를 가져옴
2. 예제
```sql
SELECT NOW(), DAYOFYEAR(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/c7b9802a-ba2f-4b38-8f77-cbb7a2972f88">
</div>

-----
### WEEK(날짜)
-----
1. 일년 중 몇 번째 주인지 가져옴
2. 예제
```sql
SELECT NOW(), WEEK(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/b3f6c6b4-4903-4aa2-b5d4-6c546749878c">
</div>

-----
### FROM_DAYS(날 수)
-----
1. 00년 00월 00일부터 날 수 만큼 지난 날짜
2. 예제
```sql
SELECT FROM_DAYS(1000)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/23ea0c63-fb73-49eb-9972-32b943d3276f">
</div>

-----
### TO_DAYS(날 수)
-----
1. 00년 00월 00일부터 날짜까지의 일 수
2. 예제
```sql
SELECT TO_DAYS(NOW())
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/2bea1f70-191a-4229-be76-33e1dadb2492">
</div>

-----
### DATE_FORMAT(날짜, 형식)
-----
1. 날짜를 형식에 맞게 만들어 반환
2. 년도 : %Y (4자리), %y (2자리)
3. 월 : %M (긴 월이름), %m (숫자의 월, [01, 02, 03..]), %b (짧은 월 이름), %c (숫자의 월 [1, 2, 3])
4. 요일 : %W (긴 요일), %a (짧은 요일), %w(0 - 일요일, 1 - 월요일 ... )
5. 일 : %D (1st, 2nd, 3rd, ...), %d(01, 02, 03, 04 ...), %e(1, 2, 3, ...), %j(일년 중 날짜, 001, 002, 003, 004, ...)
6. 시 : %l(12시간제, [1, 2, 3, ...]), %k (24시간제, 0, 1, 2, 3, ...), %h (12시간제, 01, 02, 03, ...), %H(24시간제, [00, 01, 02, ...]), %I(12시간제, [01, 02, 03, ...])
7. 분 : %i (00, 01, ...)
8. 초 : %S (00, 01, 02, ...), %s(0, 1, 2, 3, ...)
9. 시간 : %r (12시간제 시간), %T(24시간제 시간)
10. 주 : %U(일요일을 기준으로 계산한 주), %u(월요일을 기준으로 계산한 주)
11. 오전 / 오후 : %p (AM / PM)

< 예제 >
```sql
SELECT NOW(), DATE_FORMAT(NOW(), '%Y년 %m월 %d일 %H시 %i분 %s초')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/a8690227-3aed-4b27-afe8-83dd483864f3">
</div>
