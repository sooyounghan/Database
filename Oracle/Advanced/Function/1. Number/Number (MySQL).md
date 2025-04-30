-----
### 숫자 함수
-----
: 숫자와 관련된 작업을 하는 함수

-----
### ABS(숫자)
-----
1. 절댓값 함수
2. 예제
```sql
SELECT ABS(100), ABS(-100)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/b35d5666-2ccb-454f-9a3d-08065bd0d7aa">
</div>

-----
### CEIL(숫자)
-----
1. 값보다 큰 정수 중 가장 작은 정수. 소수점 이하 올림
2. 예제
```sql
SELECT CEIL(10.1), CEIL(10.4), CEIL(10.5), CEIL(10.8)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/ff05894c-6db9-4010-b5df-e86814fba420">
</div>

-----
### FLOOR(숫자)
-----
1. 값보다 작은 정수 중 가장 큰 정수. 소수점 이하 버림
2. 예제
```sql
SELECT FLOOR(10.1), FLOOR(10.4), FLOOR(10.5), FLOOR(10.8)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/04800de6-440a-4e99-bbeb-9da147d1484d">
</div>

-----
### ROUND(숫자, 자릿수)
-----
1. 자릿수를 기준으로 반올림
2. 예제
```sql
SELECT ROUND(10.1), ROUND(10.4), ROUND(10.5), ROUND(10.8)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/ba4ad8a7-645a-41c1-a1ca-d3734a297743">
</div>

```sql
SELECT ROUND(166.555, 0), ROUND(166.555, 1), ROUND(166.555, -1)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/edf02988-b8c2-4e95-bb8d-ba0307419ee8">
</div>

-----
### TRUNCATE(숫자, 자릿수)
-----
1. 자릿수를 기준으로 버림
2. 예제
```sql
SELECT TRUNCATE(166.555, 0), TRUNCATE(166.555, 1), TRUNCATE(166.555, -1)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/97b801d2-ac82-475d-b7be-6227eb5d1d0d">
</div>

-----
### POW(X, Y) 또는 POWER(X, Y)
-----
1. X의 Y승
2. 예제
```sql
SELECT POW(10, 2)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/ff064230-4f1e-47ff-abf4-a352d4a31cb5">
</div>

-----
### MOD(분자, 분모)
-----
1. 분자를 분모로 나눈 나머지를 구함
2. 예제
```sql
SELECT MOD(10, 3)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/91ac38e9-f172-41e4-b92c-d584e771c6c7">
</div>

-----
### GREATEST(숫자1, 숫자2, 숫자3, ...)
-----
1. 주어진 숫자 중 가장 큰 값 반환
2. 예제
```sql
SELECT GREATEST(10, 4, 20, 1)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/6eb22a75-0368-47b9-b1dd-ecf8946673a3">
</div>

-----
### LEAST(숫자1, 숫자2, 숫자3, ...)
-----
1. 주어진 숫자 중 가장 작은 값 반환
2. 예제
```sql
SELECT LEAST(10, 4, 20, 1)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/5f7140ac-ebe5-4c24-8c50-c92a8771b9d2">
</div>

-----
### 예제
-----
: 사원들의 사원번호와 급여를 가져옴. 단, 급여는 10% 인상된 급여를 가져오며, 소수점 이하는 올림, 버림, 반올림 값 모두 가져옴
<div align="center">
<img src="https://github.com/sooyounghan/HTTP/assets/34672301/0e40a760-bc24-459f-bc81-edbda4d45f61">
</div>
