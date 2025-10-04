-----
### 문제와 풀이
-----
1. 문제1 : 특정 열 조회 및 별칭 사용
   - products 테이블에 있는 모든 상품의 이름(name)과 가격(price) 정보를 조회
   - 단, 조회 결과의 열 이름은 각각 '상품명'과 '판매가'로 표시되어야 함
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/b07c8145-96af-45a0-9c71-5fd884ad0a68">
</div>

   - 정답
```sql
SELECT name AS 상품명, price AS 판매가
FROM products;
```

2. 문제 2 : 간단한 조건으로 필터링 하기
   - customers 테이블에서 '장영실' 고객의 모든 정보를 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/96df3911-755b-4195-8363-b9a7222ad929">
</div>

   - 정답
```sql
SELECT *
FROM customers
WHERE name = '장영실';
```

3. 문제 3 : 복합 조건으로 필터링하기 (AND, OR)
   - products 테이블에서 가격(price)이 10000원 이상이면서, 동시에 재고(stock_quantity)가 50개 미만인 상품을 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/58a11b9d-272b-4a15-b6d8-5e3faed8398c">
</div>

   - 정답
```sql
SELECT *
FROM products
WHERE price >= 10000 AND stock_quantity < 50;
```

4. 문제 4 : 특정 범위 및 목록으로 필터링하기 (BETWEEN, IN)
   - products 테이블에서 product_id가 2번, 3번, 4번 중 하나에 해당하는 상품들의 이름(name)과 가격(price)을 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/5fa0ab14-61f9-4d1d-a95f-c6830fc957ff">
</div>

   - 정답 : IN 연산자를 사용하여 특정 목록에 포함되는 데이터를 조회
```sql
SELECT name, price
FROM products
WHERE product_id IN (2, 3, 4);
```

5. 문제 5 : 문자열 패턴으로 검색하기 (LIKE)
   - customers 테이블에서 주소(address)가 '서울특별시'로 시작하는 고객의 이름(name)과 전체 주소(address)를 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/f8f8c075-3ce2-43f4-b5ea-e8c21093b807">
</div>

   - 정답 : LIKE 와 와일드카드 % 를 사용하여 '서울특별시'로 시작하는 모든 주소를 찾음
```sql
SELECT name, address
FROM customers
WHERE address LIKE '서울특별시%';
```

6. 문제 6 : NULL 값 데이터 조회하기 (IS NULL)
   - products 테이블에서 상품 설명(description)이 아직 입력되지 않은(NULL) 상품의 모든 정보를 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/e7a376dc-3dce-4a24-8626-67021ef20ba5">
</div>

   - 정답 : NULL 값을 비교할 때는 등호(=)가 아닌 IS NULL 연산자를 사용
```sql
SELECT *
FROM products
WHERE description IS NULL;
```

7. 문제 7 : 결과 정렬하기 (ORDER BY)
   - products 테이블의 모든 상품 정보를 가격(price)이 비싼 순서(내림차순)대로 정렬하여 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/1fb773eb-ee7d-4a7f-bd3a-826271a4067e">
</div>

   - 정답 : ORDER BY 절에 정렬 기준 열을 명시하고, 내림차순을 위해 DESC 키워드를 사용
```sql
SELECT *
FROM products
ORDER BY price DESC;
```

8. 문제 8 : 다중 기준으로 정렬하기
   - products 테이블의 상품 정보를 먼저 가격(price)의 오름차순으로 정렬하고, 만약 가격이 같다면 재고 수량(stock_quantity)이 많은 순(내림차순)으로 정렬하여 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/83af882d-e400-4b44-acc7-97dd7391a5f6">
</div>

   - 정답 : ORDER BY 절에 1차 정렬 기준(price ASC)과 2차 정렬 기준(stock_quantity DESC)을 콤마로 구분하여 순서대로 나열
```sql
SELECT *
FROM products
ORDER BY price ASC, stock_quantity DESC;
```

9. 문제 9 : 조회 결과 개수 제한하기 (LIMIT)
   - customers 테이블에서 가장 최근에 가입한 고객 2명의 모든 정보를 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/2451e362-8c61-4c6a-a950-95f171abcaa4">
</div>

   - 정답 : 가입일( join_date )을 내림차순( DESC )으로 정렬하여 최신 가입자 순으로 만든 뒤, LIMIT를 이용해 상위 2개의 결과만 가져옴
```sql
SELECT *
FROM customers
ORDER BY join_date DESC
LIMIT 2;
```

10. 문제 10 : 중복 값 제거하여 조회하기 (DISTINCT)
   - orders 테이블을 참조하여, 한 번이라도 주문을 한 적이 있는 고객의 ID(customer_id)와 주문한 상품의 ID(product_id) 조합을 중복 없이 조회
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0a92f7b0-ff6a-4c08-8f81-e3aad6427639">
</div>

   - 정답 : SELECT 절에 DISTINCT 키워드를 사용하여 customer_id 와 product_id 의 유일한 조합을 조회
```sql
SELECT DISTINCT customer_id, product_id
FROM orders;
```

11. 문제 11 : 종합 실전 문제
    - products 테이블에서 가격이 3000원을 초과하고 재고가 100개 이하인 상품들을 대상으로, 재고가 많은 순서대로 정렬하여 상위 3개의 상품명과 재고 수량을 조회
    - 이때 상품명은 '상품 이름'으로, 재고 수량은 '남은 수량'으로 출력
    - 실행 결과 (별칭에 공백( ) 같은 특수문자를 사용할 때는 백틱(`)으로 감싸면 됨)
<div align="center">
<img src="https://github.com/user-attachments/assets/62e162ec-3f63-4608-831a-3a6bbca3b08e">
</div>

   - 정답 : WHERE로 조건을 필터링하고, ORDER BY로 정렬한 뒤, LIMIT로 개수를 제한하며, SELECT에서는 AS를 사용해 별칭을 지정
```sql
SELECT
     name AS `상품 이름`,
     stock_quantity AS `남은 수량`
FROM
     products
WHERE
     price > 3000 AND stock_quantity <= 100
ORDER BY
     stock_quantity DESC
LIMIT 3;
```
