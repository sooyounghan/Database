-----
### NULL - 알 수 없는 값
-----
1. NULL 비교
   - 현재 보급형 스마트폰의 description 값이 NULL
   - description 이 NULL 인 상품을 = 비교를 통해 조회
```sql
SELECT * FROM products WHERE description = NULL
```
  - 실행 결과 : 없음
  - 분명 보급형 스마트폰 의 description 값은 NULL 이기 때문에 조회가 될 것으로 기대했는데, 기대와는 다르게 아무 상품도 조회되지 않음

2. NULL 연산 - 알 수 없음
   - 데이터베이스에서 '값이 없음'을 나타내는 특별한 상태를 NULL
   - NULL은 숫자 0 이나 빈 문자열('')과는 완전히 다른 개념 : 말 그대로 '알 수 없는 값', '존재하지 않는 값'
   - 💡 NULL 은 특정 값이 아니기 때문에 등호( = )로 비교할 수 없음
     + 이는 SQL에서 NULL 이 '값이 없는 상태'를 의미하는 특별한 존재이기 때문임
     + NULL 은 0도 아니고, 공백 문자열('')도 아닌, '알 수 없는 값'이라는 개념에 가까움

3. 💡 따라서 어떤 값 = NULL 이라는 비교 연산은 항상 '알 수 없음(UNKNOWN)'이라는 결과를 반환
   - NULL = NULL 조차도 참(TRUE)이 아닌 '알 수 없음(UNKNOWN)'
   - 비교는 양쪽이 다 값을 가질 때만 참 거짓을 결정할 수 있음
   - WHERE 절은 조건의 결과가 '참(TRUE)'인 행만 반환하므로, '알 수 없음(UNKNOWN)'으로 판별된 행은 결과에 포함 시키지 않음
   - 이런 문제를 해결하기 위해 SQL은 IS NULL 이라는 특별한 키워드를 제공
   - NULL 인지 아닌지를 검사하기 위해서는 반드시 IS NULL 또는 IS NOT NULL을 사용해야 함
     + IS NULL : 해당 열의 값이 NULL 인 행을 찾음
     + IS NOT NULL : 해당 열의 값이 NULL 이 아닌, 즉, 데이터가 입력된 행을 찾음

4. products 테이블에서 상품 설명(description)이 없는(NULL) 상품 조회하기
```sql
SELECT * FROM products WHERE description IS NULL;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/5a3312e0-bf2a-4631-8217-3527489b8c78">
</div>

   - 반대로 설명이 있는 상품만 찾으려면 IS NOT NULL 을 사용
```sql
SELECT * FROM products WHERE description IS NOT NULL;
```

   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/499d8269-7600-433d-b618-a7539aeac97f">
</div>

   - 이처럼 IS NULL 연산자를 사용해야만 NULL 상태인 데이터를 정확히 찾아낼 수 있음
   - 💡 NULL을 비교할 때는 =이 아니라 IS를 사용해야 함

5. NULL 정렬
   - NULL은 문자도, 숫자도 아닌데 도대체 어디에 위치해야 하는가?
   - ORDER BY 는 이 NULL 을 어떻게 처리하는가/
   - 💡 MySQL의 NULL 정렬 규칙 : 결론부터 말하자면, MySQL은 NULL을 가장 작은 값으로 취급
      + ORDER BY column ASC (오름차순 ): NULL 값이 가장 먼저 나옴 (가장 작은 값으로 취급)
      + ORDER BY column DESC (내림차순) : NULL 값이 가장 나중에 나옴 (가장 작은 값이라 맨 아래로 밀려남)
   - 이것은 데이터베이스 시스템마다 정책이 다를 수 있으므로(Oracle은 NULL을 가장 큰 값으로 취급), 사용하는 DB가 어떤 규칙을 따르는지 명확히 아는 것이 중요
   -  description 열을 오름차순으로 정렬하기
```sql
SELECT * FROM products ORDER BY description ASC;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/719dcbc2-30de-4542-b59a-7475483797c6">
</div>

   - NULL 값은 가장 작은 값 : 예상대로 description 이 NULL 인 '보급형 스마트폰'이 가장 먼저 나옴
   - description ASC로 정렬했으므로 ㅈ → ㅊ → ㅍ 순서로 정렬
   - description 열을 내림차순으로 정렬하기
```sql
SELECT * FROM products ORDER BY description DESC;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0e16ef30-69e4-49e4-969b-5c1b19afb42b">
</div>

   - 내림차순으로 정렬하니, 가장 작은 NULL 값이 가장 마지막에 위치하는 것을 확인할 수 있음
   - description DESC로 정렬했으므로 ㅍ → ㅊ → ㅈ 순서로 정렬

6. 실무 팁 : NULL 위치를 강제로 바꾸고 싶을 때
   - NULL은 가장 작은 값이기 때문에 내림차순으로 정렬하면 마지막에 출력
   - 이때는 ORDER BY 절에 조건을 추가하는 트릭을 사용 : IS NULL 을 활용하는 것
```sql
SELECT product_id, name, description, description IS NULL
FROM products
ORDER BY description DESC;
```
   - 앞서 사용한 쿼리의 SELECT에 description IS NULL을 추가
   - 실행 결과 
<div align="center">
<img src="https://github.com/user-attachments/assets/06cd1146-2b34-4ef3-8caf-e61f67b16d89">
</div>

   - description IS NULL 필드 : description IS NULL 조건에 만족하는 경우에는 1을 반환하며, 나머지는 0을 반환
   - description DESC로 정렬했으므로 ㅍ → ㅊ → ㅈ 순서로 정렬
   - 💡 MySQL은 TRUE는 1, FALSE는 0이라는 숫자로 표현
   - 💡 (열_이름 IS NULL) 이라는 조건은 해당 열의 값이 NULL 이면 1(TRUE)을, NULL 이 아니면 0(FALSE)을 반환
   - 💡 이를 이용해서 정렬
```sql
SELECT product_id, name, description, description IS NULL
FROM products
ORDER BY description IS NULL DESC;
```
   - description IS NULL을 정렬 조건에 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/80798c94-98e9-4b16-9cab-8d43be3aad4a">
</div>

   - description IS NULL을 만족하는 보급형 스마트폰의 값이 1이고, 나머지는 그보다 작은 0
   - 이 순서로 내림차순 정렬
   - 원하는 NULL 값이 가장 먼저 출력
   - 💡 그런데, 현재 상품은 description 순서가 아니라 description IS NULL 순서로 정렬되어 있는 것 : 이럴 때 앞서 배운 다중 열 정렬을 사용
```sql
SELECT product_id, name, description, description IS NULL
FROM products
ORDER BY description IS NULL DESC, description DESC;
```
   - 💡 description IS NULL DESC 을 첫 번째 정렬 조건으로 description DESC 을 두 번째 정렬 조건으로 사용
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/a9650d70-251b-49d2-9b6f-173e30afbedc">
</div>

   - 1순위인 description IS NULL 에서 보급형 스마트폰의 값만 1이므로, 따라서 가장 상단에 출력
   - 나머지 값들은 description IS NULL 의 값이 모두 0이므로, 1순위 값이 0으로 같으므로 2순위인 description DESC 를 통해 순서를 조정
   - 여기서는 내림차순으로 정렬했으므로 ㅍ → ㅊ → ㅈ 순서로 정렬
   - ORDER BY (description IS NULL) DESC 라는 1차 정렬 기준 덕분에 NULL 값이 (결과 1)이 NULL이 아닌 값들(결과 0)보다 먼저 오게 됨
   - 그리고 description DESC라는 2차 정렬 기준으로 NULL 이 아닌 값들끼리 정렬된 것을 볼 수 있음
   - 이런 식으로 NULL의 위치를 자유자재로 제어할 수 있음
