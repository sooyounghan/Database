-----
### 데이터베이스 시작하기
-----
1. 데이터의 집합, 데이터베이스 생성과 선택
   - 가장 먼저 할 일은 데이터를 담을 거대한 저장 공간, 즉 '데이터베이스'를 만드는 것
   - 쇼핑몰의 모든 데이터를 보관할 전용 창고를 짓는 것과 같음

2. CREATE DATABASE : 데이터베이스 생성하기
    - CREATE DATABASE 는 말 그대로 데이터베이스라는 창고를 만드는 명령어
    - 쇼핑몰의 이름을 '마이샵(my_shop)'이라고 가정하고, 이 이름으로 데이터베이스를 생성
```sql
CREATE DATABASE my_shop;
```
   - 데이터베이스는 세미콜론(;) 단위로 하나의 SQL로 인식
   - 따라서 실행할 SQL의 마지막에 ;를 넣어주면 됨

3. 명령을 실행하면, MySQL 서버 안에 my_shop이라는 이름의 독립된 데이터 공간이 만들어짐
    - 고객, 상품, 주문 테이블들을 만들어 나갈 것
   
4. MySQL 워크벤치 쿼리 실행 방법
   - MySQL 워크벤치에서 메뉴바 Query에 보면 다양한 쿼리 관련 기능이 제공
     + Execute (All or Selection) : 영역을 지정하지 않으면 화면에 있는 모든 SQL을 다 실행
     + 영역을 지정하면 지정한 영역의 SQL만 실행
     + Execute Current Statement : 현재 마우스 커서가 있는 SQL만 실행

   - 쿼리를 실행하면 하단의 Action Output에 결과가 출력
     + 초록색은 성공, 빨간색은 실패 : 실패한 경우 메시지를 확인하면 어떤 이유로 실패했는지 알 수 있음

5. 참고 : 대소문자 구분
   - CREATE DATABASE 와 같이 SQL이 제공하는 기본 키워드들은 대소문자를 구분하지 않음
   - my_shop 같이 직접 만든 이름은 데이터베이스 종류나 환경 설정에 따라 대소문자를 구분하기도 함
   - 따라서 다음과 같이 사용 가능 : create database my_shop;

6. 실무 팁
   - 데이터베이스 이름은 보통 프로젝트 이름이나 서비스의 특징을 나타내는 것으로 정함
   - 소문자와 언더스코어(_)를 조합하여 만드는 것이 일반적인 관례 (예) my_shop , shopping_mall_db)

7. USE : 작업할 데이터베이스 선택하기
   - 서버에는 my_shop 데이터베이스 외에도 다른 여러 데이터베이스가 존재할 수 있음
   - 따라서 앞으로 수행할 모든 작업(테이블 생성, 데이터 추가 등)이 정확히 어느 데이터베이스를 대상으로 하는지 알려줘야 함
   - USE 명령어는 비유하자면 여러 개의 엑셀 파일 중 앞으로 작업할 파일 하나를 더블 클릭해서 여는 행위와 같음
```sql
USE my_shop;
```

   - 이 명령을 실행하면, 지금부터 우리가 입력하는 모든 SQL 명령어는 my_shop 데이터베이스 안에서 실행
   - 데이터 베이스를 생성한 후에는 항상 USE 명령어로 작업 공간을 선택하는 것을 잊지 말아야 함
   - MySQL 워크벤치를 다시 시작하면 반드시 USE my_shop 명령어를 다시 실행해서 데이터베이스를 선택해야 하며, 그렇지 않으면 데이터베이스가 선택되지 않았다는 다음과 같은 오류가 발생
```
Error Code: 1046. No database selected Select the default DB to be
used by double-clicking its name in the SCHEMAS list in the sidebar.
```

-----
### 데이터를 담을 표, 테이블 생성
-----
1. CREATE TABLE : 테이블 설계하고 만들기
   - CREATE TABLE은 테이블의 구조를 정의하고 생성하는 명령어
   - 어떤 열(Column)들로 구성될지, 각 열에는 어떤 종류의 데이터가 들어갈지를 명시해야 함
```sql
CREATE TABLE sample (
 product_id INT PRIMARY KEY,
 name VARCHAR(100),
 price INT,
 stock_quantity INT,
 release_date DATE
);
```

2. 이렇게 하면 my_shop 데이터베이스 안에 상품 정보를 저장할 sample 테이블이 만들어짐
3. 이렇게 테이블을 만드는 과정을 '테이블을 설계(또는 정의)한다'고 말함
4. SQL 코드 해석
   - "sample 라는 이름의 테이블을 생성하며, 이 테이블은 product_id (정수), name (최대 100자 문자열), price(정수), stock_quantity (정수), release_date (날짜) 라는 5개의 열로 구성"
   - 데이터베이스 테이블에는 반드시 기본 키(PRIMARY KEY)가 있어야 함
     + 기본 키란, 테이블에 있는 모든 행(Row)들 중에서 특정 행 하나를 유일하게 식별할 수 있는 열(Column) 또는 열들의 조합
     + 여기서 기본 키(PRIMARY KEY)는 product_id 컬럼을 지정
     + 따라서 product_id에는 항상 다른 값이 입력되어야 함
     + 우리는 product_id를 통해서 이곳에 입력되는 값을 명확히 구분할 수 있음

5. 데이터 타입 간단 소개 : 각 열에 저장될 데이터의 종류를 알려주는 '데이터 타입(Data Type)' 
  - INT : 정수(Integer)를 의미
     + 1, 100, -50과 같은 소수점이 없는 숫자를 저장할 때 사용
     + 상품의 가격, 재고 수량, 고객 ID 등에 적합
   - VARCHAR(n) : 문자열(Variable Character)을 의미
     + n은 저장할 수 있는 최대 글자 수
     + 예를 들어 VARCHAR(100)은 최대 100글자까지의 텍스트를 저장할 수 있다는 뜻
     + 고객 이름, 상품명처럼 길이가 다양한 글자를 저장할 때 사용
   - DATE : 날짜(YYYY-MM-DD 형식)를 저장할 때 사용
     + 주문일, 가입일, 상품 출시일 등을 기록하기에 적합

-----
### 구조 확인과 삭제
-----
1. DESCRIBE : 테이블 구조 확인하기
    - DESCRIBE 명령어는 테이블이 어떤 열들로, 어떤 데이터 타입으로 구성되어 있는지 그 구조를 보여줌
    - DESCRIBE는 DESC 로 줄여서 사용할 수 있음
 ```sql
 DESC sample; -- 또는 DESCRIBE sample;
 ```

2. 주석
   - SQL 앞에 --와 공백을 넣으면 주석이 되며, 주석은 실행되지 않음
   - 주석은 컴퓨터가 아닌 사람이 이해할 용도의 메모
   - 이 명령을 실행하면 방금 우리가 만든 sample 테이블의 설계도(열 이름, 데이터 타입 등)를 표 형태로 깔끔하게 보여줌

3. 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/aa1cf348-e79f-461f-9bfe-7eded55a13dd">
</div>

4. SHOW DATABASES, SHOW TABLES : 목록 보기
   - 현재 DBMS 서버에 어떤 데이터베이스들이 있는지, 그리고 현재 선택된 데이터베이스(my_shop) 안에 어떤 테이블들이 있는지 목록을 보고 싶을 때 사용
   - 현재 서버에 있는 모든 데이터베이스 목록을 보여주며, my_shop도 보일 것
```sql
SHOW DATABASES;
```
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/396acbc9-f669-4685-9d2e-bd5ca624f055">
</div>

   - 실행 결과는 환경에 따라 조금씩 다를 수 있음 (my_shop은 꼭 있어야 함)
   - 나머지는 DBMS 운영을 위해 내부에서 사용하는 기본 데이터베이스들

   - 이번에는 my_shop 데이터베이스에 있는 테이블 목록을 확인
```sql
SHOW TABLES;
```

  - 반드시 USE my_shop 을 먼저 실행한 후에 SHOW TABLES SQL을 실행해야 함
  - my_shop 안에 있는 테이블 목록을 보여줌 : 앞서 우리가 만든 sample 테이블이 보임
<div align="center">
<img src="https://github.com/user-attachments/assets/e63b7e5b-6ffb-4e8a-bcf1-cd099a5b64f8">
</div>

5. DROP TABLE , DROP DATABASE : 삭제하기
   - 때로는 실수로 만들거나 더 이상 필요 없는 테이블이나 데이터베이스를 삭제해야 할 때가 있음
   - DROP 명령어는 구조 자체를 완전히 삭제하므로 사용에 매우 신중해야 함
   - sample 테이블을 그 안의 모든 데이터와 함께 영구적으로 삭제
```sql
DROP TABLE sample;
```
   - SHOW TABLES; 로 사라진 테이블을 확인
   - 아무런 데이터도 조회되지 않을 것
<div align="center">
<img src="https://github.com/user-attachments/assets/532ad1d5-a325-4da7-914c-4f646fd4647d">
</div>

   - 이번에는 my_shop 데이터베이스를 삭제
   - 그 안의 모든 테이블, 데이터와 함께 영구적으로 삭제
```sql
DROP DATABASE my_shop;
```
   - SHOW DATABASES; 로 사라진 데이터베이스를 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/50808676-1ce3-44b2-a226-77d6fa926e6d">
</div>

   - my_shop 데이터베이스가 제거된 것을 확인할 수 있음

6. 주의 : 실제 운영 중인 서비스에서 DROP 명령어, 특히 DROP DATABASE는 절대 함부로 사용해서는 안 됨
   - 모든 데이터가 사라지는 돌이킬 수 없는 재앙을 초래할 수 있음
   - 삭제 전에는 항상 백업이 되어 있는지, 그리고 정말 삭제해야 하는 대상이 맞는지 두 번, 세 번 확인하는 습관을 들여야 함

-----
### 예제를 위해 데이터베이스와 테이블 다시 생성
-----
```sql
-- 데이터베이스 생성
CREATE DATABASE my_shop;

-- 데이터베이스 선택
USE my_shop;

-- 테이블 생성
CREATE TABLE sample (
 product_id INT PRIMARY KEY,
 name VARCHAR(100),
 price INT,
 stock_quantity INT,
 release_date DATE
);
```

1. 메뉴바 Query Execute(All or Selection)을 선택하면 모든 쿼리를 한 번에 실행할 수 있음
  - 이때 마우스로 범위를 지정하면 지정한 범위의 쿼리만 실행
  - 마우스로 범위를 지정하지 말고 실행

2. SHOW TABLES; 로 테이블이 잘 생성되었는지 다시 확인
3. 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/f9954c37-b79e-44f0-bfd4-8109914b41a1">
</div>

-----
### 데이터베이스의 기본, CRUD 맛보기
-----
1. 테이블 안에 실제 데이터를 넣고(Create), 읽고(Read), 수정하고(Update), 삭제하는 (Delete) 방법
   - 이 네 가지 기본 작업을 묶어서 CRUD라고 부르며, 데이터 관리의 가장 핵심적인 기능
   
2. INSERT : 데이터 넣기 (Create)
   - INSERT 는 테이블에 새로운 행(Row)을 추가하는 명령어
   - sample 테이블에 신상품을 하나 등록
```sql
INSERT INTO sample (product_id, name, price, stock_quantity, release_date)
VALUES (1, '프리미엄 청바지', 59900, 100, '2025-06-11');
```
   
   - 이 구문은 "sample 테이블의 product_id, name, price, stock_quantity, release_date 열에 각각 1 , '프리미엄 청바지' , 59900 , 100 , '2025-06-11' 값을 넣어서 새로운 행을 추가해라" 라는 의미
   - 숫자는 그냥 사용하면 되고, 문자열과 날짜는 작은따옴표(') 로 감싸주어야 함

   - 실행 결과
```
1 row affected (0.00 sec)
```
   - 1개의 행이 저장되었다는 뜻

3. SELECT : 데이터 꺼내보기 (Read)
   - SELECT 는 테이블에 저장된 데이터를 조회하는 명령어
   - sample 테이블에 있는 모든 데이터를 보고 싶다면 * (별표, asterisk)를 사용
   - * 는 '모든 열(컬럼)'을 의미
```sql
SELECT * FROM sample;
```
   - SELECT : 원하는 열(컬럼)을 선택
   - FROM : 원하는 테이블을 선택
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/2602dc68-8083-4702-987a-ef27d2050409">
</div>

   - 이 명령은 sample 테이블의 모든 열과 모든 행을 보여줌
   - 방금 우리가 INSERT 한 '프리미엄 청바지' 데이터가 보일 것
   - 만약 상품의 이름과 가격만 보고 싶다면, 보고 싶은 열의 이름을 SELECT 다음에 직접 지정하면 되며, 이때 쉼표(,)로 각각의 열(컬럼)을 구분
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/f990878e-65f4-4e34-b1e9-19e2f1f23c48">
</div>

4. UPDATE : 데이터 바꾸기 (Update)
   - UPDATE 는 이미 저장된 데이터의 내용을 수정하는 명령어
   - '프리미엄 청바지'의 가격을 낮추어서 40000원으로 변경
```sql
UPDATE sample
SET price = 40000
WHERE product_id = 1;
```
   - 이 구문은 "sample 테이블에서, product_id 가 1 인 행을 찾아서, 그 행의 price 값을 40000 으로 변경해라"라는 의미
   - UPDATE : 데이터를 변경할 테이블을 지정
   - SET : 변경할 열(컬럼)과 그 값을 지정
   - WHERE : 변경할 행(로우)을 선택
   - 실행 결과
```
1 row(s) affected Rows matched: 1 Changed: 1 Warnings: 0
```
   - WHERE 조건에 맞는 1개 행(Rows matched: 1)을 찾아 그중 1개 행의 데이터를 변경(Changed: 1)했음을 의미
   - SELECT * FROM sample; 로 다시 조회해 보면 가격이 40000원으로 변경된 것을 확인
   - 실행 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/85fcb438-2d2a-4e23-a5bf-62d0a535b26c">
</div>

5. DELETE : 데이터 지우기 (Delete)
   - DELETE 는 테이블에서 특정 행을 삭제하는 명령어
   - '프리미엄 청바지' 상품을 단종시켜서 삭제
 ```sql
 DELETE FROM sample
 WHERE product_id = 1;
 ```
   - 이 구문은 " sample 테이블에서, product_id 가 1 인 행을 삭제해라" 라는 의미
   - DELETE FROM : 삭제할 테이블을 지정  
   - WHERE : 삭제할 행(로우)을 선택
   - 실행 결과
```
1 row affected (0.00 sec)
```
   - 1개의 행이 성공적으로 삭제되었음을 의미

   - 삭제 확인 : SELECT * FROM sample; 로 다시 조회해 보면 데이터가 사라진 것을 확인할 수 있음
```
0 row(s) returned
```

-----
### 참고 : 데이터베이스의 대소문자 구분
-----
1. SELECT, FROM, CREATE 같은 SQL 키워드들은 대소문자를 구분하지 않음
   - 따라서 select, SELECT, SelecT 는 모두 같은 뜻
2. 하지만 우리가 직접 지정한 테이블 명, 컬럼 명은 DBMS 종류나 서버 환경, 또는 설정에 따라 대소문자를 구분하기도 함
   - 따라서 sample, SAMPLE이 다른 결과를 보일 수 있음
   - 이러한 혼란을 피하기 위해 실무에서는 보통 같은 규칙을 따름

3. SQL 키워드는 대문자로, 이름은 소문자로, 즉, SELECT, FROM, CREATE 같은 SQL 예약어는 대문자로 작성하고, products, customer_id와 같이 우리가 직접 짓는 테이블 / 열 이름은 소문자로 작성하는 것이 일반적인 관례(convention)

   - 이 방식의 장점은 가독성이 좋다는 것
   - 하지만 이렇게 하면 SQL을 작성할 때 대소문자를 중간에 계속 변환해야 하기 때문에 실무에서는 그냥 모두 소문자를 사용하는 경우도 많음

4. 이름은 소문자와 언더스코어(_) 조합으로, 즉, my_shop, order_details 처럼 단어를 구분하는 언더스코어를 사용해 이름을 지음
   - 이렇게 하면 어떤 상황에도 일관되게 동작하는 코드를 작성할 수 있음
   - 데이터 자체는 대소문자를 구분 : SELECT * FROM customers WHERE name = 'kim' 과 SELECT * FROM customers WHERE name = 'KIM' 은 다른 결과가 나옴

5. 💡 결론적으로, "어떤 환경에서든 동일하게 작동하는 코드를 만들기 위해, 데이터베이스와 테이블, 열 이름은 항상 소문자로 작성하는 습관을 들이는 것이 가장 좋음" : 이 원칙만 지키면 대소문자 문제로 골치 아플 일은 없을 것
