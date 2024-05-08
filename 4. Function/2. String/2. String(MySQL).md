-----
### 문자열 함수
-----
: 컬럼에 저장되어 있는 문자열에 대한 작업을 할 수 있는 함수

-----
### CONCAT(문자열1/컬럼1, 문자열2/컬럼2, 문자열3/컬럼3, ...) 
-----
1. 문자열을 합치는 함수
2. 예제
```sql
SELECT CONCAT('aaa', 'bbb', 'ccc')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/0a6467ba-6aa5-4865-bbd9-4d70ec828cc5">
</div>

-----
### INSERT(문자열/컬럼, 시작 위치, 길이, 새로운 문자열)
-----
1. 문자열의 '시작위치부터 길이만큼 문자열을 제거'하고, 그 자리에 새로운 문자열을 삽입
   - 시작 인덱스 : 1
3. 예제
```sql
SELECT INSERT('aaaaa', 2, 2, 'bbb')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/4d8c4812-b2bf-402c-863c-7711e901ce7b">
</div>

```sql
SELECT INSERT('aaaaa', 2, 0, 'bbb')
FROM DUAL;
```
  - 2번 위치에서부터 0만큼의 길이의 문자열을 제거하고, 삽입
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/282eb206-210a-4d82-ac75-c4ad2862d6a4">
</div>

-----
### REPLACE(문자열, 기존문자열, 새로운 문자열) 
-----
1. 문자열에서 기존 문자열을 찾아 제거하고, 그 자리에 새로운 문자열을 삽입
2. 예제
```sql
SELECT REPLACE('aabbcc', 'bb', 'ff')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/0d3288aa-888a-4dfc-854f-f32a7f316a3b">
</div>

-----
### INSTR(문자열1, 문자열2)
-----
1. 문자열1에서 문자열2를 찾아 위치를 반환. 위치는 1부터 시작하며 문자열2를 찾지 못하면, 0을 반환
2. 예제
```sql
SELECT INSTR('abcdefg', 'cde')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/071d6b8c-1141-4617-84f4-547b83e4dbad">
</div>

3. 없는 문자열 검색 시 0을 반환
```sql
SELECT INSTR('abcdefg', 'kk')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/0ca4d910-17b8-4a9b-b5c7-26d75ad0c3f5">
</div>

-----
### LEFT(문자열, 개수)
-----
1. 문자열의 좌측부터 개수만큼 가져옴
2. 예시
```sql
SELECT LEFT('abcdefg', 3)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/ac3c8e7e-0485-454c-8d82-fd8efa142dff">
</div>

-----
### RIGHT(문자열, 개수)
-----
1. 문자열의 우측부터 개수만큼 가져옴
2. 예시
```sql
SELECT RIGHT('abcdefg', 3)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/533db487-7344-44e4-8656-95135ce4e3bd">
</div>

-----
### MID(문자열, 시작위치, 개수)
-----
1. 문자열에서 시작위치에서 개수만큼 가져옴
2. 예시
```sql
SELECT MID('abcdefg', 3, 3)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/45f2a0cf-11c9-4eb5-8554-23386e7ed6be">
</div>

-----
### SUBSTRING(문자열, 시작위치, 개수)
-----
1. 문자열의 시작위치에서 개수만큼 가져옴
2. 예시
```sql
SELECT SUBSTRING('abcdefg', 3, 3)
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/59aa41cb-ffc9-43df-8667-a1d1d7c42286">
</div>

-----
### LTRIM(문자열)
-----
1. 문자열의 좌측 공백을 제거
2. 예시
```sql
SELECT CONCAT('[', LTRIM('        abc         '), ']')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/0b068662-c5c2-481c-80ec-e8d66650fbb6">
</div>

-----
### RTRIM(문자열)
-----
1. 문자열의 우측 공백을 제거
2. 예시
```sql
SELECT CONCAT('[', RTRIM('        abc         '), ']')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/11fb698d-df9c-4c62-9306-71e6646732b9">
</div>

-----
### TRIM(문자열)
-----
1. 문자열의 좌/우측 공백을 제거
2. 예시
```sql
SELECT CONCAT('[', TRIM('        abc         '), ']')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/990f805d-cb31-48ac-85fc-e214749688b9">
</div>

-----
### LCASE(문자열), LOWER(문자열)
-----
1. 문자열을 모두 소문자로 변경
2. 예시
```sql
SELECT LCASE('abCDef'), LOWER('abCDef')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/7cb10779-2a1a-4302-937b-dd94ca48dc3d">
</div>

-----
### UCASE(문자열), UPPER(문자열)
-----
1. 문자열을 모두 대문자로 변경
2. 예시
```sql
SELECT UCASE('abCDef'), UPPER('abCDef')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/7673ed81-9073-4d22-bf36-d12b452f20d7">
</div>

-----
### REVERSE(문자열)
-----
1. 문자열을 반대로 가져옴
2. 예시
```sql
SELECT REVERSE('abcdef')
FROM DUAL;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/24183ec8-3280-4047-8519-c9fc3c07205a">
</div>

-----
### 문자열 함수 예제
-----
1. 사원의 이름을 성과 이름을 하나의 문자열로 가져옴
```sql
SELECT CONCAT(FIRST_NAME, ' ', LAST_NAME)
FROM EMPLOYEES;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/c7398212-3005-459a-abfd-a54d7c5f0a60">
</div>

2. 단, 모두 소문자로 가져옴
```sql
SELECT LOWER(CONCAT(FIRST_NAME, ' ', LAST_NAME))
FROM EMPLOYEES;
```
<div align="center">
<img src="https://github.com/sooyounghan/Java/assets/34672301/1d582205-c0f8-4199-b950-3bd50bcedb90">
</div>
