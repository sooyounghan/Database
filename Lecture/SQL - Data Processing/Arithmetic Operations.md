-----
### 산술 연산
-----
1. 산술 연산이 필요한 이유
   - 쇼핑몰의 products 테이블에는 상품의 가격(price)과 재고 수량(stock_quantity)이 각각 저장되어 있음
   - 재고 자산 총액은 가격 * 재고 수량 으로 계산할 수 있음
   - SELECT 문으로는 각 상품의 가격과 재고를 따로 조회할 수밖에 없음
```sql
SELECT name, price, stock_quantity
FROM products;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/3719cef1-c65e-413b-8a71-0bdb9b5c4ff2">
</div>

  - SQL 안에서 직접 계산하는 기능이 필요

2. SELECT절 안에서 사칙연산 활용하기
   - SELECT 문으로 컬럼을 조회할 때, 숫자 타입의 컬럼이라면 우리가 아는 사칙연산(+, -, *, /)을 그대로 사용할 수 있음
   - 각 상품의 재고 자산 총액을 계산해보자. price 컬럼과 stock_quantity 컬럼을 곱하면 됨
```sql
SELECT name, price, stock_quantity, price * stock_quantity
FROM products;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/73ee3b68-f992-45f5-8618-87ec437c2c48">
</div>

   - 데이터베이스가 각 상품별로 price와 stock_quantity를 곱한 결과를 새로운 컬럼으로 만들어서 보여줌
   - 그런데 price * stock_quantity라는 컬럼 이름이 너무 길고 보기 좋지 않음
   - 이럴 때는 앞서 배운 것처럼 별칭(Alias)을 사용해서 깔끔한 이름을 붙여줄 수 있음 : AS 키워드를 사용
```sql
SELECT
     name,
     price,
     stock_quantity,
     price * stock_quantity AS total_stock_value
FROM
     products;  
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/93b0693e-78fe-44a4-ba47-b29d88ca1be8">
</div>

  - 실무에서는 이렇게 계산된 컬럼에 별칭을 붙이는 것이 좋음

3. 다양한 산술 연산 소개
   - 덧셈 (+) : 모든 상품 가격에 배송비 3000원을 더해서 '예상 구매 가격'을 계산해볼 수 있음
```sql
SELECT name, price, price + 3000 AS expected_price
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a4d03fa5-2fc5-4d4c-8068-9873f96bad85">
</div>

   - 뺄셈 (-): 모든 상품을 1000원 할인 판매한다고 가정하고 할인된 가격을 계산할 수 있음
```sql
SELECT name, price, price - 1000 AS discounted_price
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/d5d6f45e-bd5d-442a-9a13-bc3b3342845d">
</div>

   - 나눗셈 (/): 상품 가격을 10으로 나눠서 10개월 무이자 할부 시 월 납부액을 계산할 수 있음
```sql
SELECT name, price, price / 10 AS monthly_payment
FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/10b3187a-a91d-4b17-9fa2-484a119c5b04">
</div>

4. 이처럼 SELECT 절에서 산술 연산을 활용하면, 단순히 저장된 데이터를 꺼내 보는 것을 넘어, 의미 있는 정보를 계산하고 가공하여 조회할 수 있음
