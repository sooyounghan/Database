-----
### 기본 제약 조건
-----
1. 제약 조건은 데이터베이스에 '쓰레기 데이터'가 들어오는 것을 막는 최후의 보루 역할
2. NOT NULL : NULL 값 방지
   - 역할 : 해당 컬럼에 NULL 값(값이 없는 상태)이 저장되는 것을 허용하지 않음, 즉, 반드시 필요한 정보가 누락되는 것을 막음
   - 문법 예시 : email VARCHAR(255) NOT NULL
   - 위반 시나리오 : 회원 가입 시 필수 정보인 이메일을 실수로 NULL로 입력하려고 시도
```sql
-- 이메일(NOT NULL)에 NULL 값을 입력 시도 (에러 발생)
INSERT INTO users (name, email)
VALUES ('냐옹이', NULL);
```
   - 실행 결과 에러
```
Error Code: 1048. Column 'email' cannot be null
```
   - 데이터베이스는 email 컬럼은 NULL 일 수 없다는 명확한 에러 메시지를 반환하며, 이 INSERT 명령을 거부
   - 이렇게 NOT NULL 제약 조건은 필수 데이터의 누락을 원천적으로 차단

2. UNIQUE : 중복 값 방지
   - 역할 : 해당 컬럼에 들어가는 모든 값은 테이블 내에서 반드시 고유해야(unique)함, 즉, 중복 데이터가 쌓이는 것을 막음
   - 문법 예시 : email VARCHAR(255) UNIQUE
   - 위반 시나리오 : 이미 가입된 이메일 주소(```'sean@example.com'```)로 다른 사람이 또 가입을 시도하는 상황
```sql
-- 이메일(UNIQUE)에 중복 값을 입력 시도 (에러 발생)
INSERT INTO users (name, email, address)
VALUES ('가짜 션', 'sean@example.com', '서울시 어딘가');
```
   - 실행 결과 에러
```
Error Code: 1062. Duplicate entry 'sean@example.com' for key 'users.email'
```
   - 데이터베이스는 ```'sean@example.com'``` 이라는 값이 users.email 키에 대해 중복되었다는 에러와 함께 INSERT를 거부

3. PRIMARY KEY (기본 키) : 행의 대표 식별자
   - 역할 : 테이블의 각 행을 고유하게 식별할 수 있는 단 하나의 대표 키로, NOT NULL과 UNIQUE 제약 조건의 특징을 모두 포함
     + 즉, 기본 키 컬럼은 절대 NULL 일 수 없으며, 절대 중복될 수 없음
   - 특징 : 테이블당 오직 하나만 설정할 수 있으며, 이 기본 키를 기준으로 데이터가 저장되고 검색되므로 성능에도 매우 중요한 역할을 함 (MySQL은 기본 키에 자동으로 고성능 인덱스를 생성)
   - 문법 예시 : user_id BIGINT PRIMARY KEY
   - 위반 시나리오 : 이미 존재하는 user_id 1번으로 새로운 데이터를 삽입하려고 하거나, id를 NULL로 삽입하려고 시도
```sql
-- 기본 키(PRIMARY KEY)에 중복 값을 입력 시도 (에러 발생)
INSERT INTO users (user_id, name, email)
VALUES (1, '누군가', 'someone@example.com');
```
   - 실행 결과 에러
```
Error Code: 1062. Duplicate entry '1' for key 'users.PRIMARY'
```
   - 참고로 users 테이블의 user_id는 AUTO_INCREMENT를 사용 : 따라서 자동 증가하는 값이기 때문에 NULL을 입력해도 값이 자동으로 입력
   - 만약 AUTO_INCREMENT 가 아니라면 PK에 NULL 을 입력하면 오류가 발생

4. DEFAULT : 기본값 설정
   - 역할 : 특정 컬럼에 값을 명시적으로 입력하지 않았을 경우, 자동으로 설정될 기본값을 지정
     + 엄밀히 말해 무결성을 '강제'하는 규칙이라기보다는, 데이터 누락을 방지하고 입력을 편리하게 해주는 기능에 가까움
   - 문법 예시 : status VARCHAR(50) DEFAULT 'PENDING'
   - 주문 테이블 DDL
```sql
CREATE TABLE orders (
   order_id BIGINT AUTO_INCREMENT,
   ...
   status VARCHAR(50) DEFAULT 'PENDING', -- PENDING, COMPLETED, SHIPPED, CANCELLED
   ...
);
```
   - 활용 시나리오 : orders 테이블에 주문을 추가할 때, 주문 상태(status)를 따로 지정하지 않으면 기본값인 'PENDING'으로 자동 설정
```sql
-- status 컬럼을 생략하고 INSERT
INSERT INTO orders (user_id, product_id, quantity)
VALUES (2, 2, 1);
```
```sql
SELECT *
FROM orders
ORDER BY order_id DESC
LIMIT 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/99b6cc7a-50b2-49f2-a44f-000bcebdb49a">
</div>

  - status 컬럼에 우리가 지정한 기본값인 'PENDING'이 자동으로 들어간 것을 확인할 수 있음

5. 지금까지 배운 제약 조건들은 모두 하나의 테이블 내부에서 데이터의 유효성을 지키는 규칙들
   - 하지만 데이터베이스의 핵심은 '관계'이며, 이 관계는 여러 테이블에 걸쳐 있음
     + orders 테이블의 user_id 는 반드시 users 테이블에 존재하는 값이어야만 함
