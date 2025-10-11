-----
### 프로시저, 함수, 트리거 - 실습
-----
1. 저장 프로시저 (Stored Procedure) 실습
   - 시나리오 : 특정 고객의 주소를 변경하고, 그 변경 이력을 logs 라는 별도의 테이블에 기록하는 두 가지 작업을 하나의 프로시저로 묶어보기
   - 실습 준비 : logs 테이블 생성
```sql
DROP TABLE IF EXISTS logs;

CREATE TABLE logs (
     id INT AUTO_INCREMENT PRIMARY KEY,
     description VARCHAR(255),
     created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

   - 프로시저 생성 (CREATE PROCEDURE)
     + MySQL 클라이언트에서 여러 줄의 명령어로 이루어진 프로시저를 만들 때는, 명령어의 끝을 알리는 구분자(delimiter)를 세미콜론(;)이 아닌 다른 기호(예: // 또는 $$ )로 잠시 변경해야 함
     + 프로시저의 BEGIN ... END 블록 안에 여러 개의 SQL 문이 세미콜론으로 끝나기 때문임
```sql
-- 구분자를 // 로 변경
DELIMITER //

CREATE PROCEDURE sp_change_user_address(
     IN user_id_param INT,
     IN new_address_param VARCHAR(255)
)

BEGIN
    -- 1. users 테이블의 주소 업데이트
    UPDATE users SET address = new_address_param WHERE user_id = user_id_param;
   
    -- 2. logs 테이블에 변경 이력 기록
    INSERT INTO logs (description) VALUES (CONCAT('User ID ', user_id_param, ' 주소 변경 ', new_address_param));

END //

-- 구분자를 다시 ; 로 원상 복구
DELIMITER ;
```
  - sp_change_user_address 프로시저는 두 개의 입력 파라미터(IN Parameter)를 받음

   - user_id_param : 주소를 변경할 사용자의 ID
   - new_address_param : 새로 변경할 주소 값

   - 프로시저의 본문(BEGIN ... END)에서는 두 가지 SQL 문이 순차적으로 실행
     + UPDATE users SET address = new_address_param WHERE user_id = user_id_param; : users 테이블에서 user_id_param에 해당하는 사용자의 주소(address)를 new_address_param 값으로 변경
     + INSERT INTO logs (description) VALUES (CONCAT('User ID ...')); : logs 테이블에 주소 변경 이력을 삽입 (CONCAT 함수를 사용하여 로그 메시지를 동적으로 생성) : 💡 이 과정은 UPDATE가 성공적으로 수행된 후에만 실행
   - 이처럼 저장 프로시저는 여러 SQL 문을 하나의 논리적인 단위로 묶어 처리할 수 있게 해줌

   - 프로시저 호출 (CALL)
     + 이제 user_id가 2번인 '네이트' 고객의 주소를 '경기도 성남시2'로 변경하는 프로시저를 호출
```sql
CALL sp_change_user_address(2, '경기도 성남시2');
```
   - 결과 확인
      + users 테이블과 logs 테이블을 조회하여 두 작업이 모두 잘 처리되었는지 확인
```sql
SELECT address
FROM users
WHERE user_id = 2;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/adaac290-fc40-47ef-961f-d70b05b0ccbd">
</div>

```sql
SELECT *
FROM logs;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/92c5aacc-5a49-4a02-8af9-1ad64fa93629">
</div>

   - 프로시저 삭제 (DROP PROCEDURE)
```sql
DROP PROCEDURE IF EXISTS sp_change_user_address;
```

2. 저장 함수 (Stored Function) 실습
   - 시나리오 : 상품의 정가(price)와 할인율(discount_rate)을 받아, 최종 판매가를 계산해서 반환하는 함수 fn_get_final_price 생성
   - 먼저 이 예시를 위한 DDL과 샘플 데이터 생성
```sql
DROP TABLE IF EXISTS stored_items;

CREATE TABLE stored_items (
     item_id BIGINT AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(255) NOT NULL,
     price INT NOT NULL,
     discount_rate DECIMAL(5, 2)
);
```
```sql
INSERT INTO stored_items (name, price, discount_rate) VALUES
('고성능 노트북', 1500000, 10.00),
('무선 마우스', 25000, 20.00),
('기계식 키보드', 120000, 30.00),
('4K 모니터', 450000, 40.00),
('전동 높이조절 책상', 800000, 50.00);
```
  - 함수 생성 (CREATE FUNCTION) : 💡 함수는 프로시저와 달리 반드시 RETURNS로 반환 타입을 명시해야 함
```sql
DELIMITER //

CREATE FUNCTION fn_get_final_price(
   price_param INT,
   discount_rate_param DECIMAL(5, 2)
)

RETURNS DECIMAL(10, 2)

DETERMINISTIC
BEGIN
     -- 최종 가격을 계산 (가격 * (1 - 할인율/100))
     RETURN price_param * (1 - discount_rate_param / 100);
END //

DELIMITER ;
```
   - fn_get_final_price 함수는 두 개의 입력 파라미터(price_param, discount_rate_param)를 받아, 최종 판매가를 계산하여 반환
   - RETURNS DECIMAL(10, 2) : 함수가 DECIMAL(10, 2) 타입의 값을 반환할 것임을 명시 (함수는 반드시 반환 타입을 지정해야 함)
   - 💡 DETERMINISTIC : 이 함수가 동일한 입력에 대해 항상 동일한 결과를 반환한다는 것을 MySQL에게 알려주는 키워드
     + 예를 들어, 2 + 3은 항상 5인 것처럼, 함수의 결과가 입력 값에만 의존하고 외부 요인(현재 시간, 랜덤값 등)에 영향을 받지 않을 때 사용
     + MySQL은 이 정보를 활용하여 쿼리 최적화에 도움을 받을 수 있음
   - RETURN price_param * (1 - discount_rate_param / 100); : 계산된 최종 가격을 반환
   - 함수는 이처럼 특정 계산을 수행하고 하나의 결과 값을 반환하는 데 특화되어 있으며, SELECT 문 등 SQL 쿼리 내에서 마치 내장 함수처럼 활용될 수 있다는 것이 가장 큰 특징

   - 함수 사용 (SELECT 문에서 호출) :  함수를 SELECT 문 안에서 사용하여, 각 상품의 최종 판매가를 계산
```sql
SELECT
     name,
     price,
     discount_rate,
     fn_get_final_price(price, discount_rate) AS final_price
FROM stored_items;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/d188e074-2258-40ea-872d-c2ca9bbb9cc0">
</div>

   - 함수 삭제 (DROP FUNCTION)
```sql
DROP FUNCTION IF EXISTS fn_get_final_price;
```

3. 트리거 (Trigger) 실습
   - 시나리오 : 고객이 탈퇴(users 테이블에서 DELETE)하기 직전에, 해당 고객의 정보를 retired_users 백업 테이블에 자동으로 저장하는 트리거
```sql
-- 본 실습을 위한 탈퇴 고객 테이블 생성
DROP TABLE IF EXISTS retired_users;

CREATE TABLE retired_users (
   id BIGINT PRIMARY KEY,
   name VARCHAR(255) NOT NULL,
   email VARCHAR(255) NOT NULL,
   retired_date DATE NOT NULL
);

-- 탈퇴 고객 데이터 입력
INSERT INTO retired_users (id, name, email, retired_date) VALUES
(1, '션', 'sean@example.com', '2024-12-31'),
(7, '아이작 뉴턴', 'newton@example.com', '2025-01-10');
```

   - 💡 트리거 생성 (CREATE TRIGGER)
     + 트리거는 BEFORE 또는 AFTER 특정 이벤트(INSERT, UPDATE, DELETE )에 대해 정의
     + OLD 는 변경 전 데이터, NEW 는 변경 후 데이터를 의미
```sql
DELIMITER //

CREATE TRIGGER trg_backup_user
BEFORE DELETE ON users
FOR EACH ROW

BEGIN
   INSERT INTO retired_users (id, name, email, retired_date)
   VALUES (OLD.user_id, OLD.name, OLD.email, CURDATE());
END //

DELIMITER ;
```
   - trg_backup_user 트리거는 users 테이블에 DELETE 이벤트가 발생하기 직전(BEFORE DELETE)에 자동으로 실행
   - BEFORE DELETE ON users : users 테이블에서 레코드가 삭제되기 직전에 트리거를 실행하라는 의미
     + BEFORE와 AFTER 키워드를 통해 이벤트 발생 시점을 지정할 수 있음
   - FOR EACH ROW : DELETE 되는 각 행에 대해 트리거의 본문(BEGIN ... END)을 한 번씩 실행하라는 의미
      + OLD.user_id , OLD.name , OLD.email : 트리거 내에서는 OLD 키워드를 사용하여 이벤트 발생 전(DELETE 의 경우 삭제될 행)의 컬럼 값에 접근할 수 있음
      + NEW 키워드는 INSERT 나 UPDATE 이벤트 시 이벤트 발생 후의 컬럼 값에 접근할 때 사용

   - CURDATE() : MySQL 내장 함수로, 현재 날짜를 반환
   - 이 트리거는 users 테이블에서 어떤 행이 삭제되든, 그 행의 user_id, name, email 정보를 retired_users 테이블에 백업하고, retired_date 에 현재 날짜를 기록
   - 개발자가 별도로 INSERT 문을 실행하지 않아도 데이터베이스 시스템이 자동으로 이 작업을 수행

   - 트리거 실행 (데이터 삭제)
      + 이제 user_id 가 5번인 '마리 퀴리' 고객을 users 테이블에서 삭제
      + 이 DELETE 문이 방아쇠가 되어 트리거가 자동으로 실행될 것
```sql
DELETE
FROM users
WHERE user_id = 5;
```
   - 결과 확인 : '마리 퀴리'가 users 테이블에서는 사라지고, retired_users 테이블에 백업되었는지 확인
```sql
SELECT *
FROM users
WHERE user_id = 5;
```
   - 실행 결과 : (0개의 행이 반환됨
```sql
SELECT *
FROM retired_users
WHERE id = 5;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5a6c8d69-5965-4eaf-8ade-a37e462eeb1d">
</div>

   - 우리가 직접 INSERT 하지 않았음에도, 트리거가 자동으로 백업 데이터를 생성한 것을 볼 수 있음
   - 트리거 삭제 (DROP TRIGGER)
```sql
DROP TRIGGER IF EXISTS trg_backup_user;
```
