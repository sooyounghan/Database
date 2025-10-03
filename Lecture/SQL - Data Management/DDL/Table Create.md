-----
### DDL - 테이블 생성
-----
1. 쇼핑몰의 요구사항
   - 고객 : 고객 id, 이름, 이메일, 비밀번호, 주소, 가입 시각이 관리되어야 함
   - 상품 : 상품 id, 이름, 설명, 가격, 재고 수량이 관리되어야 함
   - 주문 : 주문 id, 주문 고객, 주문 상품, 주문 수량, 주문 시각, 주문 상태가 관리되어야 함
     + 주문이 등록되면 최초의 주문 상태는 주문접수 상태
     + 예제를 단순화 하기 위해 한 번의 주문 시에 한 종류의 상품만 주문할 수 있음
     + 한 종류의 상품을 여러 개 주문하는 것은 가능

 2. 고객 테이블 ( customers )
```sql
CREATE TABLE customers (
     customer_id INT AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(50) NOT NULL,
     email VARCHAR(100) NOT NULL UNIQUE,
     password VARCHAR(255) NOT NULL,
     address VARCHAR(255) NOT NULL,
     join_date DATETIME DEFAULT CURRENT_TIMESTAMP
);
```
   - customer_id : 고객을 식별하는 기본 키(PRIMARY KEY), AUTO_INCREMENT로 자동 번호가 부여
     + AUTO_INCREMENT 덕분에 데이터를 저장할 때마다 값이 1씩 자동으로 증가
     + 따라서 모든 행을 구분할 수 있음
  
   - name, email, password, address : 모두 필수 값이므로 NOT NULL을 설정
   - email : 고객마다 유일해야 하므로 UNIQUE 제약 조건을 추가
   - join_date : 가입 시각. 값을 따로 넣지 않으면 DEFAULT 설정에 따라 현재 시각(CURRENT_TIMESTAMP)이 자동으로 기록

3. 날짜와 기본값 설정 옵션
```sql
CREATE TABLE test (
     ...
     created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
   - 이 기능을 사용하면 행이 추가되거나 변경된 날짜를 편리하게 관리할 수 있음
   - DEFAULT CURRENT_TIMESTAMP : 새로운 데이터 행(row)이 추가될 때, 해당 컬럼에 별도의 값을 지정하지 않으면 현재의 날짜와 시간이 자동으로 입력
   - DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP : 새로운 데이터 행(row)이 추가될때는 물론이고, 같은 행의 컬럼 값이 변경되어 업데이트될 때, 이 컬럼의 값은 현재 날짜와 시간으로 자동 갱신

4. 상품 테이블 ( products )
```sql
CREATE TABLE products (
     product_id INT AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(100) NOT NULL,
     description TEXT,
     price INT NOT NULL,
     stock_quantity INT NOT NULL DEFAULT 0
);
```
   - product_id : 상품을 식별하는 기본 키, AUTO_INCREMENT를 사용
   - price : 금액이므로 INT 타입을 사용했고, 필수 값이므로 NOT NULL
   - stock_quantity : 재고 수량으로, 필수 값이지만, 값을 넣지 않으면 기본적으로 0이 되도록 DEFAULT를 설정
   - description : 상품 설명은 길 수 있으므로 TEXT 타입을 사용, 필수는 아니므로 NOT NULL 제약 조건을 걸지 않음

5. 주문 테이블 ( orders )
```sql
CREATE TABLE orders (
     order_id INT AUTO_INCREMENT PRIMARY KEY,
     customer_id INT NOT NULL,
     product_id INT NOT NULL,
     quantity INT NOT NULL,
     order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
     status VARCHAR(20) NOT NULL DEFAULT '주문접수',

     CONSTRAINT fk_orders_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
     CONSTRAINT fk_orders_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```
   - order_id : 주문을 식별하는 기본 키, AUTO_INCREMENT를 사용
   - customer_id : 가장 중요한 부분, 이 주문을 한 고객이 누구인지를 나타냄
      + 이 값은 customers 테이블의 customer_id를 참조
      + 이 곳을 확인하면 고객의 이름을 찾을 수 있음  
   - product_id : 가장 중요한 부분, 어떤 상품을 주문했는지 나타냄
      + 이 값은 products 테이블의 product_id를 참조
      + 이 곳을 확인하면 상품명을 찾을 수 있음

6. 외래 키 제약조건
   - 외래 키 제약조건은 다음과 같은 명세를 가짐
```sql
CONSTRAINT [제약조건_이름]
FOREIGN KEY ([자식_테이블의_컬럼명])
REFERENCES [부모_테이블명]([부모_테이블의_컬럼명])
[ON DELETE 옵션] [ON UPDATE 옵션]
```
   - fk_orders_customers
     + CONSTRAINT fk_orders_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
     + orders 테이블의 customer_id를 customers 테이블의 customer_id와 연결하는 외래 키 설정
     + 이로써 두 테이블 사이에 '관계'가 형성
     + 이제 존재하지 않는 고객 ID로 주문을 넣으려는 시도는 데이터베이스 단에서 차단
     + 외래 키 제약조건의 이름에는 관계 있는 테이블의 참조 순서와 이름을 사용 (orders -> customers 이렇게 하면 이름으로 관계를 쉽게 파악할 수 있음)

   - fk_orders_products
     + CONSTRAINT fk_orders_products FOREIGN KEY (product_id) REFERENCES products(product_id)
     + orders 테이블의 product_id를 products 테이블의 product_id와 연결하는 외래 키 설정
     + 이로써 두 테이블 사이에 '관계'가 형성
     + 이제 존재하지 않는 상품 ID로 주문을 넣으려는 시도는 데이터베이스 단에서 차단
     + 외래 키 제약조건의 이름에는 관계 있는 테이블의 이름을 사용해서 fk_orders_products라고 설정하여, 이름으로 관계를 쉽게 파악할 수 있다 (orders -> products)
