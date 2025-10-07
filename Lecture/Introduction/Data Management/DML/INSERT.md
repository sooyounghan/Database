-----
### DML - 등록
-----
1. INSERT는 테이블에 새로운 데이터 행(row)을 추가하는 명령어
   - INSERT 문의 기본 문법
```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```
   - table_name : 데이터를 추가할 테이블의 이름
   - (column1, column2, ...) : 값을 지정할 열의 목록 (이 목록을 생략하면 테이블의 모든 열에 순서대로 값을 넣어야 함)
   - VALUES (value1, value2, ...) : 지정된 각 열에 삽입될 값으로, 열 목록과 값 목록은 순서와 개수가 정확히 일치해야 함

2. 방법 1 : 모든 열(컬럼) 추가하기
   - 가장 기본적인 방법은 데이터를 추가할 열의 목록을 생략하는 것
   - 그리고 열의 순서대로 모든 값을 (VALUES)에 제공하는 것
   - customers 테이블에 새로운 회원을 가입
```sql
INSERT INTO customers VALUES (NULL, '강감찬', 'kang@example.com', 'hashed_password_123', '서울시 관악구', '2025-06-11 10:30:00');

INSERT INTO customers VALUES (NULL, '이순신', 'lee@example.com', 'hashed_password_123', '서울시 관악구', '2025-06-12 10:30:00');
```
   - customer_id 에 NULL 을 넣은 이유 : 이 열은 AUTO_INCREMENT 로 설정했기 때문에, NULL을 명시하거나 아예 열 목록에서 생략하면 MySQL이 알아서 다음 순번의 숫자를 자동으로 채워줌
   - join_date에 직접 날짜와 시간을 입력
   - 실행 결과
```sql
SELECT * FROM customers;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/16fc5357-8e58-406e-8f96-43266b6cbe3f">
</div>

   - 실무에서 password 는 암호화해서 저장해야 함

3. 방법 2 : 원하는 특정 열만 골라서 데이터 추가하기
   - 모든 열에 데이터를 넣을 필요가 없을 때, 예를 들어 선택 입력 항목이 있을 때 사용하는 방식
   - 예를 들어 AUTO_INCREMENT나 DEFAULT 값이 설정된 열은 굳이 명시할 필요가 없음
   - 이들을 생략하면 코드가 훨씬 간결해짐
```sql
INSERT INTO customers (name, email, password, address)
VALUES ('세종대왕', 'sejong@example.com', 'hashed_password_456', '서울시 종로구');
```
   - 실행 결과
```sql
SELECT * FROM customers;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c5fde8fc-680a-47fb-88fc-0360e5ecddb8">
</div>

   - 위 코드를 실행하면 customer_id 는 자동으로 다음 번호가 부여되고, join_date 는 DEFAULT 값인 현재 시각 (CURRENT_TIMESTAMP)이 자동으로 기록
   - products 테이블에 신상품을 등록하는 예시 : description (상품 설명) 열은 필수(NOT NULL)가 아니었음
```sql
INSERT INTO products (name, price, stock_quantity) VALUES ('베이직 반팔 티셔츠', 19900, 200);

INSERT INTO products (name, price, stock_quantity) VALUES ('초록색 긴팔 티셔츠', 30000, 50);
```
   - 실행 결과
```sql
SELECT * FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/86afc6a6-f3b2-4fbd-bc1b-32016bae8a9c">
</div>

   - 이렇게 하면 product_id는 자동으로 생성되고, name, price, stock_quantity에는 지정된 값이 들어감
   - 값을 지정하지 않은 description 열에는 NULL 값이 자동으로 들어감
   - 이처럼 필요한 데이터만 골라서 넣는 것이 실무에서 가장 흔하게 사용하는 방식

4. 실무 이야기 : 열 목록을 사용
   - 열 목록을 생략하고 INSERT INTO products VALUES (NULL, '베이직 반팔 티셔츠', ...) 와 같이 값만 순서대로 나열하는 방법은 추천하지 않음
   - 나중에 테이블 구조가 변경(예) ALTER TABLE로 열 추가)되면 기존의 INSERT 코드는 모두 오류를 일으키기 때문임
   - 항상 데이터를 추가할 열의 목록을 명시적으로 작성하는 것이 안전하고 좋은 습관

5. 방법 3 : 한 번에 등록하기
```sql
INSERT INTO products (name, price, stock_quantity) VALUES
('검정 양말', 5000, 100),
('갈색 양말', 5000, 150),
('흰색 양말', 5000, 200);
```

   - 다음과 같이 VALUES에 여러 항목을 한 번에 넣어서 하나의 쿼리로 많은 데이터를 등록할 수 있음
   - 중간에 쉼표(,)를 사용해서 데이터를 구분하면 됨
   - 실행 결과
```sql
SELECT * FROM products;
```

<div align="center">
<img src="https://github.com/user-attachments/assets/540042e5-fca8-4c7b-92ec-0cda6aa22e5f">
</div>
