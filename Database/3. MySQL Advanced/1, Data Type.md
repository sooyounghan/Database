-----
### MySQL의 데이터 형식
-----
1. 숫자 데이터 형식
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/ebb38bf7-bf99-4682-9124-fd233a8cff72">
</div>

  - A. DECIMAL 데이터 형식은 정확한 수치를 저장
  - B. FLOAT, DOUBLE은 근사치의 숫자를 저장 (대신, 상당한 큰 숫자를 저장할 수 있음)
  - C. 소수점이 들어간 실수를 저장하려면 되도록 DECIMAL 사용하는 것이 바람직
  - D. MySQL은 부호 없는 정수를 지원 (UNSIGNED 예약어를 뒤에 붙여줌
    + TINYINT : 0 ~ 255
    + SMALLINT : 0 ~ 65535
    + MIDIUMINT : 0 ~ 16777215
    + INT : 0 ~ 약 42억
    + BIGINT : 0 ~ 1800 경
    + FLOAT, DOUBLE, DECIMAL도 가능

2. 문자 데이터 형식
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/6b6a3413-3d80-4ffb-b622-6ad21b8a8f7a">
</div>

  - A. CHAR 형식은 고정길이 문자형으로 자릿수 고정
    + 예) CHAR(100)에 'ABC' 3글자만 저장해도 100자리를 모두 확보한 후 앞 3자리를 사용하고, 97자리는 낭비
  - B. VARCHAR 형식은 가변길이 문자형
    + 예) VARCHAR(100)에 'ABC' 3글자를 저장할 경우, 3자리만 사용하여 공간을 효율적 운영 가능
    + 그러나 CHAR형식으로 설정하는 것이 INSERT/UPDATE 시에 일반적으로 더 좋은 성능 발휘
    + MySQL은 기본적으로 CHAR, VARCHAR 모두 UTF-8 형태를 지니므로, 입력한 글자가 영문, 한글 등에 따라 내부적으로 크기가 달라짐
  - C. 참고로 MySQL의 기본 문자 세트(Defulat Character Set)는 my.ini 또는 my.cnf 파일에 기본적으로 설정
```
# 클라이언트 문자 세트
[mysql]
default-character-set=UTF-8

# 서버 문자 세트
[mysqld]
character-set-server=UTF-8
```

  - D. BINARY와 VARBINARY는 바이트 단위 이진 데이터 값을 저장하는데 사용
  - E. TEXT 형식은 대용량 글자를 저장하기 위한 형식으로 필요 크기 따라 TINYTEXT, TEXT, MEDIUMTEXT, LONGTEXT 등 형식으로 사용 할 수 있음
  - F. BLOB(Binary Long Object)은 사진, 동영상 파일, 문서 파일 등의 대용량 이진 데이터를 저장하는데 사용
      + BLOB도 필요한 크기에 따라 TINYBLOB, BLOB, MEDIUMBLOB, LONGBLOB 등의 형식 사용 가능
  - G. ENUM은 열거형 데이터를 사용할 때 사용될 수 있음
      + 예) 요일(일, 월, 화, 수, 목, 금, 토)을 ENUM 형식으로 설정 가능
  - H. SET은 최대 64개를 준비한 후 입력은 그 중 2개씩 세트 데이터를 저장시키는 방식 사용

3. 날짜와 시간 데이터 형식
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/d4f5bf77-1096-45dc-9375-34e13cbe909e">
</div>

  - 날짜와 시간형 데이터의 예
```sql
SELECT CAST('2020-10-19 12:35:29.123' AS DATE) AS 'DATE';
SELECT CAST('2020-10-19 12:35:29.123' AS TIME) AS 'TIME';
SELECT CAST('2020-10-19 12:35:29.123' AS DATETIME) AS 'DATETIME';
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/fb3f7fd8-6718-44a5-8c15-3faa350a3bf9">
</div>

4. 기타 데이터 형식
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/ebfc476f-eff9-4369-85bd-d38b9404e40c">
</div>

-----
### LONGTEXT, LONGBLOB
-----
1. MySQL은 LOB(Large Object, 대량의 데이터)을 저장하기 위해 LONGTEXT, LONGBLOB 데이터 형식 지원
2. 지원되는 데이터의 크기는 약 4GB 크기의 파일을 하나로 데이터로 저장할 수 있음
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/31135afc-67cf-4b1e-a876-5246a2cb5a52">
</div>

-----
### 변수의 사용
-----
1. SQL 또한 변수(Variable)를 선언하고 사용 가능
2. 변수 선언과 값의 대입 형식
```sql
SET @변수이름 = 변수의 값; -- 변수의 선언 및 값 대입
SELECT @변수이름; -- 변수 값 출력
```

3. 스토어드 프로시저나 함수 안에서 변수를 사용하는 방법은 DECLARE문으로 선언한 후 사용 가능
   - 또한, 스토어드 프로시저나 함수 안에서 @변수명 형식이 아닌 변수명만 사용
   - 즉, @변수명 : '전역 변수' / DECLARE 변수명은 스토어드 프로시저나 함수 안에서 '지역 변수'처럼 사용

4. Workbench를 재시작할 때 까지 계속 유지되지만, 종료하면 소멸

5. 변수 사용 예
```sql
USE sqldb;

SET @myVar1 = 5;
SET @myVar2 = 3;
SET @myVar3 = 4.25;
SET @myVar4 = '가수 이름 ==> ';

SELECT @myVar1;
SELECT @myVar2 + @myVar3;

SELECT @myVar4, name FROM userTbl WHERE height > 100;
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/a0b13a7f-b8da-4e0c-b278-f8fe6804915f">
</div>

  - 변수의 값은 SELECT ~ FROM문과 같이 사용 가능

  - LIMIT에는 원칙적으로 변수를 사용할 수 없으나 PREPARE와 EXECUTE문을 활용해 변수 활용 가능
```sql
@SET @myVal1 = 3;

PREPARE myQuery
FROM 'SELECT name, height FROM userTbl ORDER BY height LIMIT ?';

EXECUTE myQuery USING @myVar1;
```
<div align="center">
<img src="https://github.com/sooyounghan/Data-Base/assets/34672301/83969495-5881-431a-8cce-4aa7c914d716">
</div>

  - LIMIT는 LIMIT 3과 같이 직접 숫자를 넣어야 하며, LIMIT @변수 형식으로 사용하면 오류 발생
  - PREPARE 쿼리이름 FROM '쿼리문'은 쿼리이름에 쿼리문을 준비만 해놓고 실행하지 않음
  - EXECUTE 쿼리이름을 만나는 순간 실행
  - EXECUTE는 USING @변수를 이용해 쿼리문에서 ?으로 처리해 놓은 부분에 대입
  - 결국, LIMIT @변수 형식으로 사용된 것과 동일한 효과
