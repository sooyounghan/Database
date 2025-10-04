-----
### WHERE - 기본 검색
-----
1. 전체 데이터가 아닌, 특정한 조건을 만족하는 데이터만 콕 집어서 보고 싶을 때가 훨씬 많은데, 이럴 때 사용하는 것이 바로 WHERE 절
2. 조건문의 시작 : WHERE 절과 비교 연산자
   - WHERE 절은 FROM 절 바로 뒤에 위치하며, 우리가 원하는 행(Row)만 걸러내는 필터 역할을 함
```sql
SELECT 열이름
FROM 테이블이름
WHERE 조건;
```
   - WHERE 절에는 '조건문'이 들어가는데, 이 조건문은 보통 '비교 연산자'를 사용하여 만들어짐
   - 가장 기본이 되는 비교 연산자
<div align="center">
<img src="https://github.com/user-attachments/assets/bcecc3a2-3714-44f9-95e8-6cff505a6da9">
</div>

   - SQL에서 문자열, 날짜 값은 작은따옴표(')로 감싸주며, 숫자는 그대로 사용

   - customers 테이블에서 이메일이 ```yisunsin@example.com```인 고객 조회하기
```sql
SELECT * FROM customers WHERE email = 'yisunsin@example.com';
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0ec88733-e40f-4ba2-a83a-4920d4948dc3">
</div>

   - 수만 건의 데이터가 있었더라도, 데이터베이스는 이메일이 정확히 일치하는 단 한 명의 고객 정보만 순식간에 찾아서 보여줄 것
  
   - products 테이블에서 가격(price)이 10000원 이상인 상품만 조회
```sql
SELECT * FROM products WHERE price >= 10000;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/fd665284-6cd0-4f64-94a7-452fe81aa0d3">
</div>

  - WHERE과 비교 연산자를 사용하면 원하는 데이터를 필터링할 수 있음

3. 더 복잡한 조건을 원한다면?
   - 여러 조건들을 조합하는 논리 연산자 : AND, OR, NOT
      + 두 개 이상의 조건을 결합하여 더 정교한 필터링을 하고 싶을 때, 논리 연산자를 사용
      + AND : 양쪽의 조건이 모두 참(True)일 때 최종적으로 참 ('그리고', '~이면서'의 의미)
      + OR : 양쪽의 조건 중 하나라도 참(True)이면 최종적으로 참 ('또는', '~이거나'의 의미)
      + NOT : 주어진 조건을 부정 ('아닌'의 의미, 뒤에서 알아볼 IN, LIKE, BETWEEN, IS NULL 등과 함께 사용)

    - 가격이 5000원 이상이면서, 재고가 50개 이상인 상품 조회하기 (AND)
```sql
SELECT * FROM products WHERE price >= 5000 AND stock_quantity >= 50;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/33573c4e-dba1-4b43-bf44-c33f27d6d2cd">
</div>

  - 에어팟은 가격이 3000원이어서 제외되고, LG 그램은 재고가 35개 밖에 없으므로 제외
  - 갤럭시, 아이폰, 그리고 새로 추가된 보급형 스마트폰이 두 조건을 모두 만족하여 조회
  - 이번에는 OR를 사용 : 가격이 20000원 이거나, 재고가 100개 이상인 상품을 찾기이므로, 둘 중 하나의 조건만 만족해도 됨
  - 가격이 20000원이거나, 재고가 100개 이상인 상품 조회하기 (OR)
```sql
SELECT * FROM products WHERE price = 20000 OR stock_quantity >= 100;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/319b39ab-821a-4e20-a354-292ef9c9d10f">
</div>

  - 'LG 그램'은 price = 20000 조건을 만족해서 선택되었고, '에어팟'과 '보급형 스마트폰'은 stock_quantity >= 100 조건을 만족해서 선택
  - OR는 여러 조건 중 하나만 만족해도 결과에 포함

  - 마지막으로 같지 않다는 의미인 !=를 사용 : 가격이 20000원이 아닌 제품을 찾기
  - 가격이 20000원이 아닌 상품 조회하기 (!=)
```sql
SELECT * FROM products WHERE price != 20000;
```

  - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/4c48a38b-0042-4d46-ad4d-e6b1da1a92d9">
</div>

   - 가격이 20000원인 LG 그램을 제외한 모든 상품이 선택
