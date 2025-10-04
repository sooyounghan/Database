-----
### WHERE - 편리한 조건 검색
-----
1. 편리한 조건 검색 : BETWEEN, IN, LIKE, IS NULL
  - 매번 = 이나 > 같은 기본 연산자만 사용하면 쿼리가 길어지고 비효율적일 수 있음
  - SQL은 더 편리한 검색을 위해 여러 유용한 연산자들을 제공

2. BETWEEN : 특정 범위에 있는 값 찾기
   - products 테이블에서 가격이 5000원 이상 15000원 이하인 상품 조회하기
```sql
SELECT * FROM products WHERE price >= 5000 AND price <= 15000;
```
```sql
SELECT * FROM products WHERE price BETWEEN 5000 AND 15000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5fe86122-a989-4a76-8abb-842531b40bf1">
</div>

   - ```>= ... AND <= ...``` 보다 더 직관적이고 SQL이 이해하기 쉬워짐
   - 💡 BETWEEN은 양 끝값을 포함
   - BETWEEN을 사용하면 같은 문제를 더 간결하게 표현할 수 있음
   - BETWEEN a AND b 구문은 'a와 b 사이의 값(a, b 포함)'을 찾아줌 : a에는 최솟값, b에는 최댓값

3. NOT BETWEEN : 특정 범위를 제외한 값 찾기
   - BETWEEN 앞에 NOT 을 붙이면 정확히 그 반대의 의미가 됨
   - products 테이블에서 가격이 5000원 이상 15000원 이하가 아닌 상품 조회하기 
```sql
SELECT * FROM products WHERE price < 5000 OR price > 15000;
```
```sql
SELECT * FROM products WHERE price NOT BETWEEN 5000 AND 15000;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/b15e11e7-dc51-4f93-ade9-dc76398827b1">
</div>

  - 이럴 때 NOT BETWEEN 을 쓰면 의도를 더 명확하게 표현할 수 있음
  - '이 범위에 속하지 않는 것'을 찾는다는 의미를 바로 알 수 있음
  - 이렇게 NOT BETWEEN 을 사용하면 특정 범위를 제외한 데이터를 간결하게 조회할 수 있음

4. IN : 목록에 포함된 값 찾기
```sql
SELECT * FROM products WHERE name = '갤럭시' OR name = '아이폰' OR name = '에어팟';
```
   - 이 요청을 OR 로 해결하려면 WHERE name = '갤럭시' OR name = '아이폰' OR name = '에어팟' 처럼 길게 써야 함
   - 만약 상품이 50개라면 OR 를 49번이나 써야 할까? 이럴 때 IN 을 사용
   - IN (목록) 구문은 괄호 안에 있는 목록 중 하나라도 일치하는 것이 있으면 선택
```sql
SELECT * FROM products WHERE name IN ('갤럭시', '아이폰', '에어팟');
```
   - OR를 여러 번 쓰는 것보다 가독성이 좋음
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/c857f179-3fc8-45d9-ba66-1862c1296f81">
</div>

5. NOT IN : 목록에 포함되지 않은 값 찾기
  - IN 의 반대도 당연히 필요
```sql
SELECT * FROM products WHERE name != '갤럭시' AND name != '아이폰' AND name != '에어팟';
```
  - 이럴 때 NOT IN 을 사용하면 훨씬 간단해진다.
  - products 테이블에서 이름이 '갤럭시', '아이폰', '에어팟'이 아닌 상품 조회하기
```sql
SELECT * FROM products WHERE name NOT IN ('갤럭시', '아이폰', '에어팟');
```
  - 특정 목록에 있는 것들을 제외하고 싶을 때 NOT IN 이 유용하게 사용
  - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/7fd2b8f8-401f-48e1-a81f-63048de8a0e7">
</div>

6. LIKE : 문자열의 일부로 검색하기 (패턴 매칭)
   - 💡 = 연산자는 문자열 전체가 정확히 일치해야만 찾을 수 있음
   - 이처럼 문자열의 일부만으로 데이터를 검색하고 싶을 때 LIKE 연산자와 '와일드카드'를 함께 사용
   - 와일드카드 문자
     + % (퍼센트) : 0개 이상의 모든 문자를 의미
        * 'sejong%' : sejong으로 시작하는 모든 문자열 (```sejong@example.com,``` sejong123 등)
        * ```'%@example.com'``` : ```@example.com```으로 끝나는 모든 문자열 (```aaa@example.com, hello@example.com``` 등)
        * '%서울%' : 서울 이라는 단어를 포함하는 모든 문자열 (수도서울, 서울에 살자, 수도 서울에 살자 등)
     + _ (언더스코어) : 정확히 한 개의 문자를 의미
        * '이_신' : '이'로 시작하고 '신'으로 끝나는 세 글자 이름 (이순신, 이방신 등, 예를 들어 이나라신은 정확히 한 개의 문자가 아니므로 탈락)
   - customers 테이블에서 이메일이 'sejong'으로 시작하는 고객 검색하기
```sql
SELECT * FROM customers WHERE email LIKE 'sejong%';
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/f3def1a7-5a83-467a-b013-66ce3de4254f">
</div>

7. NOT LIKE : 특정 패턴을 제외하고 검색하기
   - 서울특별시에 살지 않는 고객 검색하기
```sql
SELECT * FROM customers WHERE address NOT LIKE '서울특별시%';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e0fe83af-700b-4465-9b16-8f174bd8b813">
</div>

   - 서울특별시로 시작하는 고객을 제외한 나머지 고객을 확인할 수 있음
