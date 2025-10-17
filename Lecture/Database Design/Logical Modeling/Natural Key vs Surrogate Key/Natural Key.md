-----
### 자연 키
-----
1. 모든 테이블에는 각 행(row)을 유일하게 식별할 수 있는 기본 키(Primary Key, PK)가 반드시 필요
2. 이 기본 키를 선택하는 과정에서 우리는 두 가지 후보, 즉 '자연 키(Natural Key)'와 '대리 키(Surrogate Key)'를 만나게 됨
   - 이 선택은 단순히 키 하나를 정하는 것을 넘어, 미래에 만들어질 시스템 전체의 유연성과 안정성을 결정하는 매우 중요한 설계 결정

3. 자연 키(Natural Key)란 무엇인가?
   - 자연 키란 이름 그대로, 우리의 비즈니스 로직 안에서 자연스럽게 발생하는, 의미를 가진 데이터를 기본 키로 사용하는 것을 말함
   - 예를 들어, 대한민국 국민 이라는 데이터가 있다면 '주민등록번호'가 자연 키가 될 수 있고, 도서 데이터에서는 'ISBN' 코드가 자연 키가 될 수 있음
   - 장점
      + 직관성 : 키 값(```test@example.com```, 19900101-000000 등) 자체에 의미가 담겨 있어, 이 값만 봐도 어떤 데이터를 가리키는지 쉽게 이해할 수 있음
      + 중복 방지 : 비즈니스 규칙상 고유해야 하는 값을 PK로 지정하므로, 데이터베이스 차원에서 값의 중복을 원천적으로 막을 수 있음 (PK는 NOT NULL + UNIQUE 제약조건)

4. 자연 키의 치명적 약점 : '변경'이라는 시한폭탄
   - 자연 키의 장점은 명확하지만, 실무에서 사용할 때 자연 키는 치명적인 약점을 가지고 있는데, 바로 '변경 가능성'
   - 현대 데이터베이스 설계의 가장 중요한 원칙 중 하나는 "기본 키는 영원히 변하지 않아야 한다(Immutable)"는 것
   - 하지만 비즈니스 로직에 종속되는 자연 키는 이 원칙을 지키기가 매우 어려움
   - 기본 키의 값은 이론적으로 변경할 수 있으며, 실제 데이터베이스에서 변경도 가능
   - 하지만 실무에서 기본 키의 값은 절대로 변경하면 안 됨

5. 기본 키의 값을 변경하면 안되는 이유
   - 시나리오 : 회원의 이메일을 PK로 사용한, 쇼핑몰이 처음 문을 열 때, "이메일은 로그인 ID이며 절대 변경할 수 없다"는 정책을 세우고 member 테이블의 PK를 email로 정했다고 가정
   - 실습 준비
```sql
-- 데이터베이스가 존재하지 않으면 생성
CREATE DATABASE IF NOT EXISTS my_shop3;

USE my_shop3;
```

   - 실습에서는 my_shop3 이라는 데이터베이스를 사용
```sql
DROP TABLE IF EXISTS orders_n;
DROP TABLE IF EXISTS member_n;

-- 잘못된 설계의 시작
CREATE TABLE member_n (
   email VARCHAR(50) NOT NULL, -- PK (자연 키)
   password VARCHAR(255) NOT NULL,
   name VARCHAR(50) NOT NULL,
   PRIMARY KEY (email)
);

-- member 테이블을 참조하는 orders 테이블
CREATE TABLE orders_n (
   order_id BIGINT NOT NULL AUTO_INCREMENT,
   member_email VARCHAR(50) NOT NULL, -- FK
   ordered_at DATETIME NOT NULL,

   PRIMARY KEY (order_id),
   CONSTRAINT fk_orders_member_n FOREIGN KEY (member_email)
   REFERENCES member_n (email)
);
```
   - 자연 키(Natural Key)를 사용하는 예시이므로 학습 목적으로 테이블을 쉽게 구분하기 위해 뒤에 _n 을 붙임
   - ERD 
     + 열쇠 : PK
     + 붉은색 마름모 : FK
     + 파란색 마름모 : 컬럼
     + 꽉 찬 마름모 : NOT NULL
     + 빈 마름모 : NULL 허용
<div align="center">
<img src="https://github.com/user-attachments/assets/b89e5d47-b162-4800-9996-07d1e835d52e">
</div>

   - 몇 달 후, 고객 지원팀에서 요청 발생 : "수 많은 회원들이 이메일을 변경하고 싶어하므로 기능을 추가"하라는 요청
      + 회사에서 긴 논의 끝에 '이메일은 로그인 ID이며 절대 변경할 수 없다'는 정책을 변경해서 이메일을 변경할 수 있게 허용
      + 회원 ```member1@old.com```이 이메일을 ```member1@new.com```으로 변경하려고 함
      + member 테이블의 PK를 직접 변경하는 UPDATE 쿼리를 실행해야 함
```sql
-- 실행하는 순간 재앙이 시작될 수 있는 쿼리
UPDATE member_n
SET email = 'member1@new.com'
WHERE email = 'member1@old.com';
```
   - 기본 키의 값을 변경하는 이 쿼리를 실행하면 시스템의 신뢰도와 안정성 자체를 파괴할 수 있는 다양한 문제들이 발생

6. 문제 1 : 참조 무결성 제약조건 위반
   - 가장 먼저 마주하는 현실적인 문제는, 앞서 실행한 UPDATE 쿼리가 애초에 성공하지 않는다는 점 : 데이터베이스가 스스로를 보호하기 위해 설정해 둔 외래 키(Foreign Key) 제약조건 때문임
   - 우리는 orders_n 테이블을 생성할 때 외래 키 제약조건을 설정 : 이는 '자식 테이블(orders_n)에서 참조하고 있는 부모 테이블(member_n)의 키 값은 변경할 수 없다'는 규칙
   - 실제로 데이터를 넣고 이메일 변경을 시도한다고 가정
     + 테스트 데이터 준비
```sql
-- 기존 이메일로 회원 가입
INSERT INTO member_n (email, password, name)
VALUES ('member1@old.com', 'hashed_password', '네이트');

-- 해당 회원이 상품 주문
INSERT INTO orders_n (member_email, ordered_at)
VALUES ('member1@old.com', NOW());
```
   - 이메일 변경 시도 및 결과 : 이제 이 회원의 이메일 변경을 시도하면, 데이터베이스는 다음과 같은 오류를 내뱉으며 쿼리 실행을 거부할 것
```sql
UPDATE member_n
SET email = 'member1@new.com'
WHERE email = 'member1@old.com';
```
   - 실행 결과
```sql
Error Code: 1451. Cannot delete or update a parent row: a foreign key
constraint fails (`my_shop3`.`orders_n`, CONSTRAINT `fk_orders_member_n`
FOREIGN KEY (`member_email`) REFERENCES `member_n` (`email`))
```
   - 이 오류 메시지는 orders_n 테이블이 fk_orders_member_n라는 외래 키 제약조건을 통해 member_n 테이블의 member1@old.com 값을 참조하고 있기 때문에, member_n 테이블의 PK를 함부로 바꿀 수 없다는 뜻
   - 이것이 바로 데이터베이스가 데이터의 정합성(Consistency)과 무결성(Integrity)을 지키는 방식
     + 만약 이 UPDATE가 성공했다면, 주문 데이터는 주인을 잃고 떠다니는 고아 데이터가 되었을 것
     + 데이터베이스의 기본 설정이 우리 시스템을 큰 재앙으로부터 보호해 준 것

7. 문제 2 : 연쇄 업데이트와 시스템 부하
   - 단 한 번의 이메일 변경으로 인해 데이터베이스 전체에 걸쳐 수천 개의 레코드를 수정하는 끔찍한 연쇄 업데이트가 발생
   - 이는 데이터베이스에 엄청난 부하 제공

8. 문제 3 : 데이터의 역사성 훼손
   - 데이터베이스가 마법처럼 모든 참조를 완벽하게 업데이트했다고 가정
   - member_n 테이블의 PK는 ```member1@new.com```이 되었고, orders_n 테이블의 member_email도 모두 ```member1@new.com```으로 바뀌었다고 가정 : 데이터의 역사성이 훼손
   - 분명히 member1 회원은 과거 ```member1@old.com```이메일을 사용하던 시점에 주문
     + 그런데 이제 데이터베이스에는 그 주문을 ```member1@new.com```이 한 것처럼 기록 : 사실 왜곡
     + 몇 달 뒤, 고객 지원팀에서 다급한 연락이 온다고 가정 : 데이터베이스의 모든 기록이 현재 시점의 이메일로 덮어씌워졌기 때문에, 과거의 특정 시점에 어떤 데이터가 유효했는지 추적할 방법이 사라진 것
   - 이는 장애 추적, 데이터 분석, 나아가 법적 분쟁 대응 등 신뢰성 있는 데이터가 필수적인 모든 영역에서 치명적인 문제

9. 문제 4 : 외부 시스템과의 연동 문제
   - 현대의 서비스는 혼자 동작하지 않음 : 쇼핑몰도 주문 데이터를 외부 배송업체 시스템, 이메일 마케팅 솔루션, 데이터 분석 플랫폼 등 수많은 외부 시스템과 연동하고 있을 것
   - ```member1@old.com```회원의 주문 정보를 외부 배송 시스템에 전달했다고 가정
     + 배송 시스템은 ```member1@old.com```을 식별자로 사용해 배송 상태를 추적하고 있을 것
     + 그런데 쇼핑몰 데이터베이스에서 이메일을 ```member1@new.com```으로 변경해 버리면, 시스템과 외부 배송 시스템 간의 데이터 동기화가 깨짐
     + 이제 두 시스템은 동일한 회원을 서로 다른 식별자(```member1@new.com vs member1@old.com```)로 인식
     + 이 상태에서 고객이 배송 상태를 조회하거나 반품을 신청하면, 우리 시스템은 ```member1@new.com```으로 배송 시스템에 정보를 요청하지만, 배송 시스템에는 해당 데이터가 존재하지 않아 오류가 발생할 것
   - 이 문제를 해결하려면 외부 시스템과 연동하는 모든 지점을 찾아 복잡한 데이터 동기화 로직을 추가로 개발해야만 함
   - 비즈니스 로직(이메일)을 PK로 사용한 대가로, 시스템 전체의 복잡도가 기하급수적으로 증가하는 것

10. 이처럼 자연 키는 '변경'이라는 시한폭탄을 품고 있음
    - 데이터베이스 내부의 정합성을 깨뜨릴 뿐만 아니라, 데이터의 역사성을 왜곡하고, 외부 시스템과의 연동까지 마비시킬 수 있는 매우 위험한 설계 방식
