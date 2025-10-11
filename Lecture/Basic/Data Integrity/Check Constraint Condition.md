-----
### CHECK 제약 조건
-----
1. 데이터의 '내용' 자체에 대한 규칙은 어떻게 적용하는 방법
   - 상품의 가격(price)이나 재고 수량(stock_quantity)은 절대 음수일 수 없음
   - 할인율(discount_rate)은 0%에서 100% 사이의 값이어야 함
   - NOT NULL 제약 조건은 가격에 -5000 이라는 값이 들어오는 것을 막지 못하며, UNIQUE 제약 조건은 할인율이 200 이 되는 것을 막지 함
   - 이런 값들은 형식적으로는 유효하지만, 비즈니스 논리상으로는 명백한 '쓰레기 데이터'

2. 이처럼 특정 컬럼에 들어갈 수 있는 값의 범위나 조건을 직접 지정하여, 한층 더 강화된 비즈니스 규칙을 적용하고 싶을 때 사용하는 것이 바로 CHECK 제약 조건
3. CHECK 제약 조건의 역할과 문법
   - CHECK 제약 조건은 특정 컬럼에 대해, INSERT 또는 UPDATE가 일어날 때마다 지정된 조건식이 '참(true)'인지를 검사
   - 만약 조건식이 거짓(false)이면, 데이터베이스는 해당 데이터의 입력을 거부하고 에러를 발생시킴
   - 문법은 테이블을 생성할 때 컬럼 정의 뒤에 CHECK (조건식) 을 추가해 주면 됨
```sql
-- 실습을 위해 기존 테이블들을 삭제한다.
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;

-- CHECK 제약 조건을 추가하여 products 테이블 재생성
CREATE TABLE products (
   product_id BIGINT AUTO_INCREMENT PRIMARY KEY,
   name VARCHAR(255) NOT NULL,
   category VARCHAR(100),
   price INT NOT NULL CHECK (price >= 0),
   stock_quantity INT NOT NULL CHECK (stock_quantity >= 0),
   discount_rate DECIMAL(5, 2) DEFAULT 0.00 CHECK (discount_rate BETWEEN 0.00 AND 100.00)
);
```

   - price >= 0 : 가격은 0 이상이어야 함
   - stock_quantity >= 0 : 재고 수량은 0 이상이어야 함
   - discount_rate BETWEEN 0.00 AND 100.00 : 할인율은 0과 100 사이의 값이어야 함 ((discount_rate >= 0 AND discount_rate <= 100)과 동일)

4. 참고 : 제약 조건에 이름을 부여하려면 DDL에서 다음과 같은 SQL을 작성
```sql
CONSTRAINT products_chk_price CHECK (price >= 0)
```

5. 위반 시나리오 1 : 가격에 음수 입력 시도
```sql
-- 가격(price)에 음수 값을 입력 시도 (에러 발생)
INSERT INTO products (name, category, price, stock_quantity)
VALUES ('오류상품', '전자기기', -5000, 10);
```
   - 실행 결과 에러
```
Error Code: 3819. Check constraint 'products_chk_1' is violated.
```

   - 데이터베이스가 products_chk_1 이라는 이름의 CHECK 제약 조건(가격은 0 이상)이 위반되었다며, 데이터 입력을 단호하게 거부

6. 위반 시나리오 2 : 할인율에 범위를 벗어난 값 입력 시도
```sql
-- 할인율(discount_rate)에 100을 초과하는 값을 입력 시도 (에러 발생)
INSERT INTO products (name, category, price, stock_quantity, discount_rate)
VALUES ('초특가상품', '패션', 50000, 20, 120.00);
```
   - 실행 결과 에러
```
Error Code: 3819. Check constraint 'products_chk_3' is violated.
```

7. CHECK 제약 조건은 애플리케이션 레벨에서 놓칠 수 있는 데이터의 유효성 검사를 데이터베이스가 직접 수행하여, 비즈니스 규칙에 어긋나는 데이터가 저장될 가능성을 원천적으로 차단하는 강력한 수단
8. 실무 : 데이터 검증, 어디서 하는 게 좋을까?
   - 요즘 대세는 애플리케이션 코드 : 대부분의 비즈니스 로직과 유효성 검사는 애플리케이션에서 직접 처리하고, 훨씬 유연하고 테스트가 쉽기 때문임
   - DB CHECK 제약 조건은 신중하게 : 이런 이유로 데이터베이스의 CHECK 제약 조건은 잘 사용하지 않는 추세
   - '최후의 방어선'으로 활용 : CHECK 제약 조건을 쓴다면, 간단하지만 절대 값이 바뀌면 안 되는 핵심 데이터에만 '최후의 방어선'으로 사용하는 것이 좋음
