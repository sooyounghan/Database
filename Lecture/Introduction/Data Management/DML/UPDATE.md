-----
### DML - 수정
-----
1. UPDATE 는 이미 존재하는 데이터의 내용을 수정하는 명령어
   - UPDATE 기본 문법
```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```
   - table_name : 데이터를 수정할 테이블의 이름
   - SET column1 = value1, ... : 수정할 열과 새로운 값으로, 쉼표(,)를 사용해 여러 열을 한 번에 수정할 수 있음
   - WHERE condition : 수정할 행을 식별하는 조건 (이 부분을 생략하면 테이블의 모든 행이 수정되므로 극도로 주의해야 함)

2. UPDATE와 SET 기본 구문
   - product_id=1 인 베이직 반팔 티셔츠를 조회
```sql
SELECT * FROM products
WHERE product_id = 1
```
   - WHERE를 사용해서 원하는 행을 지정할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/e6000d24-3c4d-43f6-97be-2ab78e72c3f3">
</div>

   - 이 상품은 '베이직 반팔 티셔츠'로, 이 상품의 가격을 9800원으로 할인하고, 재고를 580개로 변경
```sql
UPDATE products
SET price = 9800, stock_quantity = 580
WHERE product_id = 1;
```
   - 실행 결과
```sql
SELECT * FROM products
WHERE product_id = 1
```
<div align="center">
<img src="https://github.com/user-attachments/assets/4cc3f8a1-d411-4cfb-937c-e3656bd4a1a7">
</div>

   - 가격은 19900 → 9800원으로, 재고는 200 → 580으로 변경된 것을 확인 가능

3. SQL 안전 업데이트 모드
   - 데이터를 변경하는 것은 생각보다 위험할 수 있음
   - 만약 "베이직 반팔 티셔츠" 상품만 특가로 판매하기 위해 가격을 990원으로 변경하려고 했는데, 실수로 WHERE를 생략할 경우
```sql
UPDATE products
SET price = 990; -- WHERE product_id = 1; -- 실수로 생략
```
   - 이러면 모든 쇼핑몰 모든 상품의 가격이 990원이 됨 
   - 실행 결과
```
Error Code: 1175. You are using safe update mode and you tried to update a
table without a WHERE that uses a KEY column. To disable safe mode, toggle
the option in Preferences -> SQL Editor and reconnect.
```
   - 다행히도 실행되지 않고, 오류가 발생
   - 오류 메시지를 보면 안전 업데이트 모드를 사용중이라고 나옴
   - 이런 실수를 방지하기 위해 MySQL은 '안전 업데이트 모드'를 제공
```sql
SELECT @@SQL_SAFE_UPDATES;
```
   - 실행 결과 : 1
   - 실행 결과 1은 안전 업데이트 모드가 활성화 상태이고, 0은 비활성화 상태
   - 💡 안전 업데이트 모드 활성화 상태에서는 데이터를 변경하거나 삭제할 때 WHERE 절에 기본 키 컬럼을 반드시 지정해야 함 (또는 LIMIT 를 사용해서 변경하거나 삭제할 데이터의 양을 조절해야 함)
   - 💡 그래서 기본 키 컬럼을 사용하지 않은 다음과 같은 SQL도 실행이 되지 않음
```sql
UPDATE products
SET price = 990
WHERE name = '베이직 반팔 티셔츠'; -- name 컬럼을 사용
```
   - 실행 결과
```
Error Code: 1175. You are using safe update mode and you tried to update a
table without a WHERE that uses a KEY column. To disable safe mode, toggle
the option in Preferences -> SQL Editor and reconnect.
```
   - 참고로 MySQL이 제공하는 안전 업데이트 모드의 기본 설정은 비활성화(0) 상태
   - 그런데 MySQL 워크벤치는 이런 실수를 방지하기 위해 MySQL에 접근할 때 안전 업데이트 설정을 활성화
   - 정리
      + MySQL 서버 (전역) : 기본적으로 OFF (0)
      + 애플리케이션 (커넥션) : 기본적으로 OFF (0)
      + MySQL 워크벤치 : 기본적으로 ON (1)
        
   - 다음 문구를 실행하면 안전 업데이트 모드의 설정을 임시로 변경할 수 있음
```sql
SET SQL_SAFE_UPDATES = 0; -- 안전 업데이트 모드 끄기

SET SQL_SAFE_UPDATES = 1; -- 안전 업데이트 모드 활성화
```

   - MySQL 워크벤치의 기본 설정을 변경하고 싶은 경우
      + 윈도우 : 메뉴바 Edit → Preference → SQL Editor 맨 아래의 Safe Updates 체크 해제
      + MAC : 메뉴바 MySQLWorkbench → Settings... → SQL Editor 맨 아래의 Safe Updates 체크 해제
   - 안전 모드를 제거하고 다시 UPDATE 문을 실행
```sql
SET SQL_SAFE_UPDATES = 0; -- 안전 모드 제거

UPDATE products
SET price = 990;
SET SQL_SAFE_UPDATES = 1; -- 안전 모드 활성화
```
   - 안전 모드를 끄고, 필요한 기능을 실행하고 나면 안전 모드를 다시 활성화해주는 것이 안전
   - 실행 결과
```sql
SELECT * FROM products;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/bf1fa41b-effc-4343-bac5-5639f6111eb4">
</div>

   - 모든 상품의 가격이 990원이 된 것을 확인할 수 있음
   - 위 명령어를 실행하는 순간, WHERE 조건이 없으므로 products 테이블의 모든 상품 가격이 990원으로 바뀌는 문제 발생
   - 특히 MySQL 워크벤치와 같이 SQL을 직접 실행할 수 있는 도구에서 SQL을 직접 다룰 때 이런 문제가 자주 발생
   - 그리고 이런 문제를 예방하기 위해 MySQL 워크벤치는 안전 모드를 기본으로 적용

   - UPDATE 나 DELETE 문을 실행하기 전에는, 반드시 동일한 WHERE 절을 사용한 SELECT 문을 먼저 실행해서 내가 변경하려는 데이터가 정확히 맞는지를 눈으로 확인하는 습관을 들여야 함
```sql
-- 1단계: 변경할 대상을 눈으로 먼저 확인
SELECT * FROM products
WHERE name = '베이직 반팔 티셔츠';

-- 2단계: 확인된 대상에 대해서만 UPDATE를 실행
SET SQL_SAFE_UPDATES = 0; -- 안전 모드 제거

UPDATE products
SET price = 19800
WHERE name = '베이직 반팔 티셔츠';

SET SQL_SAFE_UPDATES = 1; -- 안전 모드 활성화
```
   - 실무에서는 SQL을 사용해서 데이터를 변경하거나 삭제할 때, 반드시 SELECT 쿼리를 먼저 수행해서 내가 수정할 데이터를 꼭 먼저 확인할 것
