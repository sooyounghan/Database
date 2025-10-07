-----
### 문자열 함수
-----
1. 문자열 함수가 필요한 이유
   - 고객 관리 페이지를 만든다고 가정
   - customers 테이블에는 고객의 이름(name)과 이메일(email)이 별도의 컬럼에 저장
```sql
SELECT name, email
FROM customers;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/4c7fff8a-eb1f-4834-8816-a5dc613d77fc">
</div>

   - 다음과 같이 이름과 이메일을 합쳐서 하나의 문자열로 보기 좋게 표시하고 싶음
```
이순신 (yisunsin@example.com)
```
2. CONCAT() 함수로 문자열 합치기
   - CONCAT은 'concatenate', 즉 '연결하다'라는 뜻의 영어 단어에서 유래
   - 이름 그대로 괄호 안에 전달된 여러 문자열을 순서대로 이어 붙여 하나의 문자열로 만들어줌
   - 함수 : 함수는 어떤 값을 받아서 특별한 처리를 하고 그 결과를 반환하는 기능으로 이해하면 됨
```
CONCAT(string1, string2, ...)
```

   - CONCAT에는 붙이고 싶은 값을 ,(쉼표)로 구분해서 넣으면 됨
   - CONCAT() 함수를 사용해서 고객의 이름과 이메일을 합쳐보기
```sql
SELECT CONCAT(name, ' (', email, ')')
FROM customers;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/d7559a36-edf6-4d2e-be1c-590dd333a286">
</div>

   - 결과를 보면, name 컬럼 값, (문자, email 컬럼 값) 문자가 순서대로 합쳐져서 우리가 원하던 형태로 출
   - 컬럼 이름이 너무 지저분하니 AS로 별칭을 붙여줌
```sql
SELECT
   CONCAT(name, ' (', email, ')') AS name_and_email
FROM
   customers;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/490e8e74-43b2-49c2-a64f-5e04558ecd0d">
</div>

   - 이제 name_and_email 이라는 깔끔한 이름으로 합쳐진 문자열을 얻을 수 있dma

3. 문자열을 다루는 기본 함수 소개
   - CONCAT_WS(separator, string1, string2, ...) : CONCAT 과 비슷하지만, 첫 번째 인자로 '구분자'를 받아 각 문자열 사이에 자동으로 넣어줌
     + WS는 'With Separator'의 약자
     + 💡 CONCAT_WS() 는 MySQL 전용 함수로, 다른 데이터베이스와 호환되지 않음
     + 예) 고객 이름과 이메일 주소, 주소를 -로 연결하기
```sql
SELECT CONCAT_WS(' - ', name, email, address) AS customer_details
FROM customers;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/e0fbdd6b-2c0c-46fd-a0c4-08f67482ebb6">
</div>

   - UPPER() / LOWER()
      + 문자열을 모두 대문자 또는 소문자로 변경
      + 이메일 검색 등에서 대소문자 구분 없이 비교해야 할 때 유용
      + 예) 이메일을 모두 대문자로 출력하기
```sql
SELECT email, UPPER(email) AS upper_email
FROM customers;
```

   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/9ef24111-55b5-4494-8fbf-481ae193a7be">
</div>

   - LENGTH(), CHAR_LENGTH()
      + LENGTH() : 문자열의 길이를 바이트 단위로 반환
      + CHAR_LENGTH() : 글자 수를 반환
      + 예) 고객 이름의 길이 확인하기
```sql
SELECT name, CHAR_LENGTH(name) as char_length, LENGTH(name) AS byte_length
FROM customers;
```
   - 실행 결과 (UTF-8 인코딩 기준 한글은 3바이트)
<div align="center">
<img src="https://github.com/user-attachments/assets/1908c1bd-ce13-4248-b5e9-5acb2d24e9d3">
</div>
