-----
### 대리 키
-----
1. 자연 키가 가진 '변경 가능성'이라는 치명적인 약점을 해결하기 위해 등장한 개념이 바로 대리 키(Surrogate Key), 다른 말로는 인조키(Artificial Key)
2. 대리 키의 핵심 아이디어 : 비즈니스 로직과 완전히 무관한, 오직 데이터를 식별하기 위한 용도로만 존재하는, 임의의 값을 기본 키로 사용하는 것
3. 대리 키의 존재 이유는 단 하나, 절대 변하지 않는다는 것
4. 대리 키는 비즈니스와 아무런 관련이 없는, 시스템이 자동으로 생성해 주는 값
   - 보통 1, 2, 3, ... 과 같이 순차적으로 증가하는 정수
   - f47ac10b-58cc-4372-a567-0e02b2c3d479 와 같이 전역적으로 고유함이 보장되는 UUID(Universally Unique Identifier)를 사용
   - UUID (Universally Unique Identifier)
     + 이름 그대로 전 세계에서 유일무이할 것으로 기대되는 값을 만들어내는 표준
     + 쉽게 이야기해서 아주 아주 큰 랜덤값이어서, 확률적으로 충돌이 거의 발생하지 않음
     + 약 1초에 10억 개의 UUID를 100년 동안 계속 만들다 보면 한 번 정도 충돌할 수 있음

5. member_n 테이블 대신에 새로운 테이블 member_s를 생성
   - member_id 컬럼을 만들고, AUTO_INCREMENT 속성을 부여해서 대리 키를 사용
```sql
DROP TABLE IF EXISTS orders_s;
DROP TABLE IF EXISTS member_s;

-- 올바른 설계
CREATE TABLE member_s (
   member_id BIGINT NOT NULL AUTO_INCREMENT, -- PK (대리 키)
   email VARCHAR(50) NOT NULL, -- 비즈니스 로직
   password VARCHAR(255) NOT NULL,
   name VARCHAR(50) NOT NULL,
   PRIMARY KEY (member_id),
   UNIQUE KEY uq_email (email) -- 고유해야 하는 자연 키는 UNIQUE
);

-- member_s 테이블을 안정적으로 참조하는 orders_s 테이블
CREATE TABLE orders_s (
   order_id BIGINT NOT NULL AUTO_INCREMENT,
   member_id BIGINT NOT NULL, -- FK
   ordered_at DATETIME NOT NULL,

   PRIMARY KEY (order_id),
   CONSTRAINT fk_orders_member_s FOREIGN KEY (member_id) REFERENCES member_s (member_id)
);
```
```sql
-- 기존 이메일로 회원 가입
INSERT INTO member_s (email, password, name)
VALUES ('member1@old.com', 'hashed_password', '네이트');
```
```sql
-- 앞서 저장한 회원 ID 임시 저장
SET @last_member_id = LAST_INSERT_ID();
```
```sql
-- 해당 회원이 상품 주문
INSERT INTO orders_s (member_id, ordered_at)
VALUES (@last_member_id, NOW());
```
   - 대리 키(Surrogate Key)를 사용하는 예시이므로 테이블을 쉽게 구분하기 위해 뒤에 _s를 붙임
   - member_s 테이블에 member_id라는 대리 키를 생성
   - orders_s 테이블은 이제 member_id FK를 사용

<div align="center">
<img src="https://github.com/user-attachments/assets/f522eabe-4213-4bad-9b59-cdc873e8c299">
</div>

6. 대리 키는 어떻게 자연 키의 문제를 해결하는가?
   - 영원히 변하지 않는 키
     + member_id는 회원이 가입하는 순서에 따라 1 , 2 , 3 ... 과 같이 자동으로 부여되는 값
     + 이 값은 비즈니스 로직과 아무런 관련이 없음
     + 회원이 이메일을 바꾸든, 이름을 바꾸든, 이사를 가든, member_id는 절대로 변하지 않음
     + 데이터베이스 설계의 가장 중요한 원칙 중 하나인 '기본 키는 불변(Immutable)해야 한다'를 완벽하게 만족

  - 손쉬운 이메일 변경
    + member1 회원의 member_id가 1 이라고 가정
    + 이 회원의 이메일을 ```member1@old.com```에서 ```member1@new.com```으로 변경하는 쿼리
```sql
UPDATE member_s
SET email = 'member1@new.com'
WHERE member_id = 1;
```
   - 이 쿼리는 member_s 테이블의 email 컬럼만 수정할 뿐, PK인 member_id는 전혀 건드리지 않음
   - 따라서 이 회원과 연결된 수천 개의 orders_s, posts, reviews 테이블의 어떤 데이터도 수정할 필요가 없음 : 연쇄 업데이트가 발생하지 않음
   - 데이터베이스에 부하를 주지 않으며, 데이터 정합성이 깨질 위험도 전혀 없음
   - 비즈니스 로직의 변경이 데이터 구조에 아무런 영향을 주지 않는, 완벽하게 느슨하게 결합된(Loosely Coupled) 이상적인 구조가 완성된 것

   - member_s 테이블 조회 : member_s 테이블을 조회해서 이메일이 잘 변경되었는지 확인
```sql
SELECT * FROM member_s;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c6f018ef-2908-44cd-8ba3-2b06c9e4633d">
</div>

   - orders_s 테이블 조회 : member_s 테이블의 email이 변경되어도 orders_s 테이블은 아무런 영향을 받지 않았다는 것을 확인
     + orders_s 테이블은 오직 member_id만 바라보고 있음
```sql
SELECT *
FROM orders_s;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/ab68860c-c490-4502-8e27-19d65ea2ef6f">
</div>

   - 결과를 보면 orders_s 테이블의 member_id는 1 로 그대로 유지되고 있으며, 우리는 이 값을 통해 여전히 '네이트' 회원의 주문이라는 사실을 명확하게 알 수 있음
   - 이처럼 대리 키를 사용하면 비즈니스 요구사항 변경에 매우 유연하고 안정적으로 대처할 수 있음

7. 비즈니스 로직의 유연성 확보
   - "이메일은 절대 변경할 수 없다"는 비즈니스 정책이 "이제부터 변경 가능하다"로 바뀌어도, 데이터베이스 설계는 아무런 영향을 받지 않음 : 비즈니스 로직과 식별자를 분리했기 때문임
   - 그럼 여기서 이메일의 중복을 막고 싶다면 어떻게 할까?
     + PK가 아닌 UNIQUE 제약조건을 email 컬럼에 걸어주면 됨
     + 이렇게 하면 PK의 역할(행의 유일한 식별)과 비즈니스 규칙(이메일 중복 방지)을 명확하게 분리하여 각각의 역할에 충실한, 유연하고 안정적인 설계를 만들 수 있음
    
8. 자연 키의 올바른 사용법 : UNIQUE 제약조건
   - PK를 대리 키에게 넘겨주고, 본연의 역할에 집중하면 됨
   - 즉, 비즈니스적으로 고유해야 하는 모든 컬럼에 UNIQUE 제약조건을 설정하는 것
   - 위의 member_s 테이블 설계에서 email에 UNIQUE 키를 설정할 때의 이점
     + 관계의 안정성 : 불변의 대리 키 member_id가 모든 외래 키 관계의 중심을 잡아줌
     + 데이터 무결성 : UNIQUE 제약조건이 email의 중복을 데이터베이스 차원에서 막아줌
     + 조회 성능 : UNIQUE 제약조건을 설정하면 해당 컬럼에 자동으로 인덱스가 생성되므로 email을 통한 조회 성능도 보장

9. 💡 현대 데이터베이스 설계의 표준 패턴 : 대리 키-PK, 자연 키-UNIQUE
   - 현대적인 데이터베이스 설계는 대리 키-PK, 자연 키-UNIQUE 방식이 사실상 표준
     + 대리 키(Surrogate Key)를 기본 키로 사용
     + 자연 키(Natural Key)에는 UNIQUE 제약조건 적용
   - 이 패턴은 관계 안정성, 데이터 무결성, 조회 성능이라는 세 가지를 모두 만족시키는 사실상의 표준 방식이며, 이 방식의 핵심은 역할의 분리
   - 기본 키(Primary Key)는 '대리 키(Surrogate Key)'
      + 역할 : 다른 테이블과의 관계를 맺는 '연결고리' 역할에만 집중
      + 구현 : order_id , member_id 처럼 비즈니스와 직접적인 관련이 없는, 시스템이 자동 생성하는 불변의 값 (주로 숫자나 UUID)을 사용
      + 이점 : 이 키는 절대 변하지 않으므로, 이 키를 참조하는 모든 외래 키(FK) 관계가 매우 안정적으로 유지

   - 자연 키(Natural Key)는 'UNIQUE 제약조건'
      + 역할 : 비즈니스 로직상 '데이터의 고유성'을 보장하는 역할을 담당
      + 구현 : email, 주민등록번호, 사용자 아이디 처럼 실제 비즈니스 의미를 가지며 중복되어서는 안 되는 컬럼에 UNIQUE 제약조건을 설정
      + 이점
        * 데이터 무결성 : 데이터베이스 시스템 차원에서 해당 값의 중복 입력을 원천적으로 차단하여 데이터의 신뢰성을 보장
        * 성능 보장 : UNIQUE 제약조건이 설정된 컬럼에는 자동으로 인덱스(Index)가 생성되어, 해당 컬럼을 조건으로 데이터를 조회할 때 빠른 속도를 보장

10. 정리
    - 관계는 불변의 대리 키 - PK로 안정적으로 맺고, 비즈니스 데이터의 고유성은 자연 키 - UNIQUE로 확실하게 보장
    - 이처럼 각 키의 역할을 명확히 분리하여 장점만 취하는 것이 현대적인 데이터베이스 설계의 핵심

11. 실무 이야기 - 비즈니스 환경은 언젠가 변함
    - 기본 키의 불변 조건을 현재는 물론이고 미래까지 충족하는 자연 키를 찾기는 쉽지 않음
    - 대리 키는 비즈니스와 무관한 임의의 값이므로 요구사항이 변경되어도 기본 키가 변경되는 일은 드뭄
    - 대리 키를 기본 키로 사용하되 주민등록번호나 이메일처럼 자연 키의 후보가 되는 컬럼들은 필요에 따라 유니크 인덱스를 설정해서 사용
