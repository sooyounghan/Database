-----
### 변환 함수
-----
1. 서로 다른 유혀의 데이터 타입으로 변환해 결과를 반환하는 함수
2. 묵시적 형변환 : 자동으로 데이터 타입 형변환
3. 명시적 형변환 : 변환함수를 이용해 직접 형변환

-----
### TO_CHAR(숫자 혹은 날짜, format)
-----
1. 숫자나 날짜를 문자로 변환해주는 함수

```sql
SELECT TO_CHAR(123456789, '999.999.999');
FROM DUAL;
```
: 123,456,789

```SQL
SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD')
FROM DUAL;
```
: 2024-03-08

2. DATE FORMAT 형식
<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/a9b91d3a-aa5b-4ed5-9342-2992720995c8">
</div>

3. 숫자 FORMAT 형식
<div align = "center">
<img src="https://github.com/sooyounghan/DataBase/assets/34672301/68293771-c820-4317-ace3-fe519b5ecd00">
</div>

-----
### TO_DATE(char, format), TO_NUMBER(char, format)
-----
1. 문자를 날짜형으로 변환하는 함수
2. TO_DATE : DATE 타입 / TO_TIMESTAMP :TIMESTAMP 형식

```sql
SELECT TO_DATE('20140101', 'YYYY-MM-DD')
FROM DUAL;
```
: 2014-01-01

```sql
SELECT TO_DATE('20140101 13:44:50', 'YYYY-MM-DD HH24:MI:SS')
FROM DUAL
```
: 2014-01-01 13:44:50
