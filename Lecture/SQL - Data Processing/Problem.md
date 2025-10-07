-----
### 문제와 풀이
-----
1. 문제 1: 상품 할인 가격 계산하기
   - products 테이블의 모든 상품에 대해 15% 할인된 가격을 계산하고, sale_price 라는 별칭으로 출력
   - 상품의 이름, 원래 가격(price), 그리고 할인가(sale_price)를 함께 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/75dc7e28-f515-4df7-a604-13711c7d76b2"">
</div>

   - 정답
```sql
SELECT
     name,
     price,
     price * 0.85 AS sale_price
FROM
     products;
```

2. 문제 2: 고객 정보 보기 좋게 합치기
   - customers 테이블을 사용하여, 각 고객의 이름과 주소를 CONCAT_WS() 함수를 이용해 '-' 로 연결
   - 결과 컬럼의 별칭은 customer_info로 지정
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/3bf7ad66-c0dd-44bd-9530-25e31f3db657">
</div>

   - 정답
```sql
SELECT
     CONCAT_WS(' - ', name, address) AS customer_info
FROM
     customers;
```

3. 문제 3 : 상품 설명이 없는 경우 처리하기
   - products 테이블에서 상품 설명(description)이 없는(NULL) 상품은, 상품 이름(name)을 대신 설명으로 사용하도록 조회
   - COALESCE() 함수를 사용하고, 결과 컬럼의 별칭은 product_display_info로 지정
   - 상품의 원래 이름과 최종적으로 표시될 정보를 함께 출력
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0264680a-8022-4d47-abef-6533a285d4ce">
</div>

   - 정답
```sql
SELECT
     name,
     COALESCE(description, name) AS product_display_info
FROM
     products;
```

4. 문제 4: 여러 후보 값 중 유효한 값 선택하기
   - products 테이블에 description(상품 설명) 컬럼과 name(상품명) 컬럼이 존재
   - 상품 정보를 표시할 때, description 값이 존재하면 그 값을 사용하고, description이 NULL이면 name 값을 사용
   - 만약 name 값도 NULL 이라면(그런 경우는 없지만 가정) '정보 없음'이라는 문구를 출력하고 싶음
   - 이 로직을 COALESCE() 함수를 사용하여 display_text 라는 별칭으로 출력
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/83f8c68d-29c3-4414-9be8-c5342f2d592d">
</div>

   - 정답
```sql
SELECT
     name,
     description,
     COALESCE(description, name, '정보 없음') AS display_text
FROM
     products;
```

5. 문제 5 : 이메일 주소 분리 및 분석하기
   - customers 테이블의 email컬럼에서 아이디 부분만 추출하고, 아이디의 글자 수를 계산
   - 아이디는 @ 앞부분의 문자열
   - SUBSTRING_INDEX()와 CHAR_LENGTH() 함수를 활용하여 user_id와 id_length라는 별칭으로 결과를 출력
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/c317bf54-efaa-4b07-9402-b3d8fdfc1047">
</div>

   - 정답
```sql
SELECT
     email,
     SUBSTRING_INDEX(email, '@', 1) AS user_id,
     CHAR_LENGTH(SUBSTRING_INDEX(email, '@', 1)) AS id_length
FROM
     customers;
```

