-----
### NULL 함수
-----
1. NULL 함수가 필요한 이유
   - products 테이블 : 상품 데이터 중에 '보급형 스마트폰'은 상품 설명(description)이 NULL 값 존재
```sql
SELECT name, description
FROM products;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/8161a756-cc54-46b4-8cdc-3f85dd379e79">
</div>

   - 만약 이 결과를 그대로 웹사이트에 표시한다면, 상품 설명 부분에 아무것도 나오지 않거나 'NULL'이라는 단어가 그대로 노출될 수 있음
   - 이는 사용자에게 좋지 않은 경험을 줌 : 차라리 '상품 설명이 없습니다.' 와 같은 안내 문구를 보여주는 것이 더 나은 선택
   - 이처럼 데이터에 존재할 수 있는 NULL 값을 그대로 노출하는 대신, 우리가 원하는 특정 값으로 대체해서 보여줘야 하는 상황이 바로 우리가 해결해야 할 문제

2. 해결책 : NULL 값을 다루는 함수들
   - MySQL은 NULL 값을 효과적으로 처리할 수 있는 함수들을 제공
   - 가장 대표적인 것이 IFNULL()과 COALESCE()

3. 💡 IFNULL(표현식1, 표현식2) : IFNULL() 함수는 두 개의 인자를 받음
   - 표현식1 : NULL 인지 아닌지 검사할 컬럼이나 값
   - 표현식2 : 표현식1이 NULL 일 경우 대신 반환할 값
   - 만약 표현식1이 NULL이 아니면 표현식1 의 값을 그대로 반환하고, NULL이면 표현식2의 값을 반환
   - 예) description 컬럼의 NULL 값을 '상품 설명 없음'으로 바꿔보기
```sql
SELECT
   name,
   IFNULL(description, '상품 설명 없음') AS description
FROM
   products;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/78e85ce0-9e3e-4e3a-b5b6-d46524c724ce">
</div>

   - 결과를 보면 '보급형 스마트폰'의 description이 더 이상 NULL 이 아니라 우리가 지정한 '상품 설명 없음'으로 대체된 것을 확인할 수 있음

4. COALESCE(표현식1, 표현식2, ...) : COALESCE() 함수는 IFNULL() 보다 조금 더 유연
   - 괄호 안에 여러 개의 인자를 전달할 수 있으며, 왼쪽부터 차례대로 확인해서 처음으로 NULL이 아닌 값을 반환
   - 모든 인자가 NULL 이면 결국 NULL 반환
   - 우리 예제처럼 단순히 NULL 일 때 다른 값 하나로 바꾸는 상황에서는 IFNULL()과 COALESCE()의 기능이 거의 동일
```sql
SELECT
     name,
     COALESCE(description, '상품 설명 없음') AS description
FROM
     products;
```
   - 위 쿼리의 실행 결과는 IFNULL() 을 사용했을 때와 완전히 같음
   - COALESCE()는 예를 들어, 상품에 short_description(짧은 설명) 컬럼과 long_description(긴 설명) 컬럼이 둘 다 있다고 가정
     + 우리는 짧은 설명이 있으면 그것을 쓰고, 없으면 긴 설명을 쓰고, 둘 다 없으면 '설명 없음'을 표시하고 싶을 수 있는데, 이럴 때 COALESCE()가 아주 유용
     + COALESCE(short_description, long_description, '설명 없음') : 이런 식으로 여러 대안을 순서대로 제시할 수 있다는 점에서 COALESCE()가 IFNULL() 보다 더 확장성이 좋음
     
