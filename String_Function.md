-----
## 문자열 함수
-----
: 컬럼에 저장되어 있는 문자열에 대해 처리를 해 값을 가져올 수 있는 함수

-----
### LOWER : 대문자를 소문자로 변환
-----
1. 문자열 ABcdEF 출력
```sql
SELECT 'ABcdEF'
FROM DUAL;
```

2. 문자열에 대해 소문자로 가져온다
```sql
SELECT LOWER('ABcdEF')
FROM DUAL;
```
: abcdef 출력

3. 사원들의 이름을 소문자로 가져온다.
```sql
SELECT LOWER(DNAME)
FROM EMP;
```

-----
### UPPER : 소문자를 대문자로 변환
-----
1. 문자열 ABcdEF을 대문자로 변환 출력
```sql
SELECT UPPER('ABcdEF')
FROM DUAL;
```
: ABCDEF 출력

2. 사원들의 이름을 가져오고, 소문자로 변환 후, 다시 대문자로 변환한다.
```sql
SELECT UPPER(LOWER(ENAME))
FROM EMP;
```

-----
### INITCAP : 첫 글자만 대문자로, 나머지는 소문자로 변환
-----
1. 첫 글자만 대문자로, 나머지는 소문자로 가져온다.
```sql
SELECT INITCAP('aBCDEF')
FROM DUAL;
```
: Abcedf 출력

2. 사원 이름을 첫 글자는 대문자로, 나머지는 소문자로 가져온다.
```sql
SELECT INITCAP(ENAME)
FROM EMP;
```

-----
### CONCAT : 문자열 합치기
-----
1. 'abc'와 'def' 두 문자열을 합친다.
```sql
SELECT CONCAT('abc', 'def')
FROM DUAL;
```
: abcdef 출력


2. 'kkk'문자열을 합치기 전에, 'abc'와 'def'문자열을 합치는데, 앞에 합친다.
```sql
SELECT CONCAT('kkk', CONCAT('abc', 'def'))
FROM DUAL;
```
: kkkabcdef 출력

2. 'zz'문자열을 합치기 전에, 3번 문자열 뒤에 합친다.
```sql
SELECT CONCAT(CONCAT('kkk', CONCAT('abc', 'def')), 'zz')
FROM DUAL;
```

3. 사원들의 이름과 직무를 다음과 같이 가져온다.
   - 사원의 이름은 000이고, 직무는 000입니다.
```sql
SELECT CONCAT(CONCAT(CONCAT('사원의 이름은' , CONCAT(ENAME, '이고, 직무는'), JOB), ' 입니다.')
FROM EMP;
```

4. || 연산자 : CONCAT 연산자와 동일
```sql
SELECT '사원의 이름은 ' || ENAME || '이고, 직무는 ' || JOB || ' 입니다.'
FROM EMP;
```

-----
### LENGTH : 문자열의 길이 / LENGTHB : 문자열의 바이트 크기)
-----
1. 영문/숫자 : 1Byte
2. 한글 : 글자당 3Byte

```sql
SELECT LENGTH('abcd'), LENGTHB('ABCD'), LENGTH('안녕하세요'), LENGTHB('안녕하세요')
FROM DUAL;
```
: 4, 4, 5, 15

-----
### SUBSTR : 문자열 잘라내기 (문자열, 인덱스), SUBSTRB : 문자열 Byte 단위로 잘라내기 (문자열, 인덱스)
-----
1. Oracle : 문자열의 첫 번째의 인덱스가 1
   
2. 예제 
```sql
SELECT SUBSTR('abcd', 3), substrb('abcd', 3)
from dual
```
: cd, cd

3. 한글의 경우
```sql
SELECT SUBSTR('안녕하세요', 3), substrb('안녕하세요', 3)
from dual
```
: 하세요, 녕하세요


4. 매개변수가 있는 경우 : SUBSTR(STR, 시작 Index, 글자 수)
```sql
SELECT SUBSTR('abcdefghi', 3, 4), SUBSTR('동해물과 백두산이', 3, 4)
FROM EMP
```
: cdef, 불과 백

-----
### INSTR : 문자열에서 해당 문자열 찾기
-----
1. INSTR(문자열, 찾고싶은 문자열)
```sql
SELECT INSTR('abcdabcdabcd', 'bc')
from dual;
```
: 해당 문자열이 처음 일치하는 인덱스 번호 출력 - 2

2. INSTR(문자열, 찾고싶은 문자열, 시작 인덱스)
```sql
SELECT INSTR('abcdabcdabcd', 'bc', 3)
from dual;
```
: 해당 문자열이 3번 인덱스 이후 인덱스 번호 출력 - 6

3. INSTR(문자열, 찾고싶은 문자열, 시작 인덱스, n번째 위치)
```sql
SELECT INSTR('abcdabcdabcd', 'bc', 3, 2)
from dual;
```
: 해당 문자열이 3번 인덱스 이후 2번째 등장하는인덱스 번호 출력 - 10

4. abcdefg 문자열에서 aaa 문자열을 찾는다.
```sql
SELECT INSTR('abcdefg', 'aaa')
FROM DUAL;
```
: 해당 문자열이 없으므로 0

5. 사원의 이름 중에 A라는 글자가 두번째 이후에 나타나는 사원의 이름을 가져온다.
```sql
SELECT ENAME
FROM EMP;
WHERE INSTR(ENAME, 'A') > 1;
```
```sql
SELECT ENAME
FROM EMP;
WHERE ENMAE LIKE '_A%';
```

-----
### LPAD, RPAD : 특정 문자열로 채우기
-----
1. LPAD('문자열', 숫자) : 숫자만큼 공간을 만들고, 왼쪽 기준 문자열을 채우고 나머지는 기본값인 공백을 채움
2. LPAD('문자열', 숫자, '채울 문자열') : 1번과 같으나 기본값 공백이 아닌 채울 문자열로 채움
3. RPAD('문자열', 숫자) : 숫자만큼 공간을 만들고, 오른쪽 기준 문자열을 채우고, 나머지는 기본값인 공백을 채움
4. RPAD('문자열', 숫자, '채울 문자열') : 3번과 같으나 기본값 공백이 아닌 채울 문자열로 채움

```sql
SELECT LPAD('문자열', 20, '_')
FROM DUAL;
```
: 문자열_________________

```sql
SELECT RPAD('문자열', 20, '_')
FROM DUAL;
```
: _________________문자열

-----
### TRIM : 공백 제거
-----
1. LTRIM : 왼쪽 공백 제거
2. RTRIM : 오른쪽 공백 제거
3. TRIM : 좌우 공백 모두 제거
   
```sql
SELECT LTRIM('              문자열              ')
FROM DUAL;
```
: 문자열                    

```sql
SELECT RTRIM('              문자열              ')
FROM DUAL;
```
:               문자열

```sql
SELECT TRIM('              문자열              ')
FROM DUAL;
```
: 문자열

-----
### REPLACE : 문자열 변경
-----
```sql
SELECT REPLACE('abcdefghi', 'kkkkkkkk')
FROM DUAL;
```
: kkkkkkkhi로 변경
