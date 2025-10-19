-----
### 일대일(1:1) 관계
-----
1. 하나의 행이 다른 테이블의 단 하나의 행과만 관계를 맺는 경우
2. 사실 1:1 관계는 1:N 관계와 비교해서 상대적으로 흔하지 않음
   - 왜냐하면 1:1 관계는 그냥 하나의 테이블로 합쳐버리는 것이 더 효율적인 경우가 많기 때문임

3. 일대일 관계를 사용하는 이유
   - 케이스 1 : 성능 최적화를 위한 분리
     + member 테이블에 회원의 상세 프로필 정보(자기소개, 개인 홈페이지 주소, MBTI 등)를 저장한다고 가정
     + 이 정보들은 데이터 크기가 클 수 있지만, 로그인이나 단순 회원 목록 조회 같은 핵심 기능에서는 거의 사용되지 않음
     + 만약 이 모든 컬럼이 member 테이블에 포함되어 있다면, 단순히 회원의 이름만 필요한 쿼리조차도 디스크에서 불필요 하게 큰 데이터를 읽어와야 하며, 이는 성능 저하의 직접적인 원인이 됨
     + 이때, 자주 쓰이는 핵심 정보와 가끔 쓰이는 부가 정보를 두 개의 테이블로 분리하여 1:1 관계를 맺으면 성능을 최적화 할 수 있음
        * member 테이블 : 자주 사용되는 핵심 정보(member_id, name)만 존재
        * member_detail 테이블 : 크기가 크고 가끔 사용되는 상세 정보를 저장

   - 케이스 2 : 보안 강화를 위한 분리
     + 회원의 정보 중에는 주민등록번호, 여권번호, 은행 계좌 정보와 같이 매우 민감한 개인정보가 있을 수 있음 : 이런 정보를 일반 회원 정보와 같은 테이블에 두는 것은 보안상 위험할 수 있음
     + 이때 민감 정보를 별도의 테이블로 분리하고, 해당 테이블에만 데이터베이스 관리자(DBA)가 특별히 엄격한 접근 권한을 설정할 수 있음 (예를 들어 특정 데이터베이스 사용자에게만 SELECT 권한을 부여하는 식)
        * member 테이블 : 자주 사용되는 핵심 정보(member_id, name)만 존재
        * member_sensitive 테이블 : 민감한 개인정보를 보관
     + 이렇게 분리하면, 일반적인 애플리케이션 로직은 member_sensitive 테이블에 접근할 필요도, 권한도 없게 만들 수 있음
     + 오직 본인인증이나 정산처럼 꼭 필요한 특정 기능에서만 제한적으로 접근하도록 제어하여 보안 수준을 높일 수 있음

   - 케이스 3 : 선택적 정보(0 or 1) 표현 및 확장
     + 모든 회원이 판매자(Seller)는 아님
     + 쇼핑몰의 회원 중 일부만 자신의 상품을 판매하는 판매자로 등록할 수 있다고 가정 : 이 판매자 정보(상호명, 사업자 등록번호, 정산 계좌 등)를 모든 member 테이블에 컬럼으로 추가하는 것은 비효율적
     + 대부분의 일반 회원에게는 이 컬럼들이 항상 NULL 값으로 채워져 공간을 낭비하기 때문임
     + 이런 경우, seller 라는 별도의 테이블로 분리하는 것을 고려할 수 있음

   - 케이스 4 : 비즈니스 모델에 따른 분리
     + 쇼핑몰 비즈니스가 성장하면서 '주문 처리'와 '배송 처리'의 책임과 역할이 명확하게 나뉜다고 생각
     + '주문'과 '배송'은 개념적으로 강력한 1:1 관계를 가지지만(하나의 주문에 하나의 배송), 관리 주체나 데이터의 라이프사이클이 다를 수 있음
     + orders 테이블 : 주문 자체에 대한 핵심 정보(주문자, 주문일, 주문 상태, 결제 금액 등)를 관리하여, 이 정보는 주로 '주문 / 결제 시스템'에서 사용
     + delivery 테이블 : 배송과 관련된 정보(배송지, 운송장 번호, 배송 상태, 배송사 등)만을 전문적으로 관리하여, 이 정보는 '물류 / 배송 시스템'에서 주로 사용
     + 테이블을 분리하면 다음과 같은 장점이 존재
        * 관심사의 분리(Separation of Concerns) : '주문'과 '배송'이라는 서로 다른 비즈니스 도메인의 데이터를 명확하게 분리하여 모델을 더 깔끔하게 이해하고 관리할 수 있음
        * 독립적인 확장성 : 나중에 배송 관련 기능(예) 합배송, 해외 배송, 실시간 위치 추적)이 고도화될 때, delivery 테이블과 관련 로직만 수정하면 되며, orders 테이블과 이를 사용하는 주문 시스템은 전혀 영향을 받지 않음
     + 이처럼, 논리적으로는 강한 연관성이 있는 1:1 관계일지라도, 비즈니스 역할과 시스템의 구조적 설계에 따라 테이블을 분리하는 것은 매우 현명하고 실용적인 설계 전략

4. 일대일 관계 만들기
   - 회원과 회원 상세 정보를 일대일 관계로 분리해서 설계
     + member 테이블 : 자주 사용되는 핵심 정보(member_id, name)만 남김
     + member_detail 테이블 : 크기가 크고 가끔 사용되는 상세 정보를 저장
   - 먼저 회원 상세 정보를 저장할 member_detail 테이블을 생성
     + member_detail 테이블은 고유의 ID인 member_detail_id 를 기본 키(PK)로 가지고, 어떤 회원의 상세 정보인지를 나타내기 위해 member 테이블을 참조하는 member_id 를 외래 키(FK)로 가짐
   - 테이블 설계 예제
```sql
DROP TABLE IF EXISTS member_detail;
DROP TABLE IF EXISTS member;

CREATE TABLE member (
   member_id BIGINT NOT NULL AUTO_INCREMENT,
   name VARCHAR(50) NOT NULL,
   PRIMARY KEY (member_id)
);

CREATE TABLE member_detail (
   member_detail_id BIGINT NOT NULL AUTO_INCREMENT, -- 회원 상세 ID (PK)
   member_id BIGINT NOT NULL, -- 회원 ID (FK)
   bio TEXT NULL, -- 자기소개
   website_url VARCHAR(255) NULL, -- 개인 홈페이지
   mbti VARCHAR(10) NULL, -- MBTI

   PRIMARY KEY (member_detail_id),
   CONSTRAINT fk_detail_member FOREIGN KEY (member_id)
   REFERENCES member (member_id)
);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/7760b100-2bab-426f-a140-31b2049fa4e0">
</div>

   - 이제 테스트를 위해 member 테이블에 '김개발'이라는 회원을 추가하고, member_detail 테이블에 해당 회원의 상세 정보를 추가
```sql
-- 테스트 회원 추가
INSERT INTO member (name) VALUES ('김개발');

-- '김개발' 회원의 상세 정보 추가 (member_id = 1)
INSERT INTO member_detail (member_id, bio, mbti)
VALUES (1, '안녕하세요, 최고의 개발자가 되고 싶습니다.', 'INTJ');
```

   - 이제 각 테이블의 데이터가 어떻게 저장되었는지, 그리고 두 테이블을 조인했을 때 어떻게 보이는지 확인
   - 먼저 member 테이블을 조회
```sql
SELECT *
FROM member;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/211f5d06-b00a-4c6f-85c0-d07e231613b5">
</div>

   - 다음으로 member_detail 테이블을 조회
```sql
SELECT *
FROM member_detail;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/0ae38543-ee53-4bf7-9398-9252869c4309">
</div>

  - 마지막으로 member 테이블과 member_detail 테이블을 JOIN 해서 필요한 정보를 함께 조회
```sql
SELECT m.member_id, m.name, d.bio, d.mbti
FROM member m
JOIN member_detail d ON m.member_id = d.member_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/686e6b1f-9fee-4ed0-a4c2-3c1336bc8f2e">
</div>

  - 그런데 만약 실수로 '김개발' 회원의 상세 정보가 한 번 더 추가된다면 어떻게 될까?
```sql
-- '김개발' 회원의 상세 정보 또 추가
INSERT INTO member_detail (member_id, bio, mbti)
VALUES (1, '취미는 코딩입니다.', 'ENTP');
```

  - 이 쿼리는 아무런 오류 없이 성공적으로 실행
  - '김개발' 회원의 상세 정보를 조회
```sql
SELECT *
FROM member_detail
WHERE member_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/51852992-bdc5-40fc-98aa-a0d30052d5fe">
</div>

  - 한 명의 회원이 단 하나의 상세 정보를 가지는 1:1 관계를 의도했지만, 현재 테이블 구조는 한 명의 회원이 여러 개의 상세 정보를 가질 수 있는 1:N 관계가 되어버리게 됨
  - member_id FK 컬럼에 중복된 값이 들어갈 수 있기 때문임

5. UNIQUE 제약조건으로 1:1 관계 보장하기
   - 이 문제를 해결하고 진정한 1:1 관계를 데이터베이스 레벨에서 강제하려면, member_detail 테이블의 member_id 컬럼에 UNIQUE 제약조건을 추가해야 함
   - 이 제약조건은 해당 컬럼에 중복된 값이 들어오는 것을 막아줌
   - 수정된 테이블 설계 예제
```sql
-- 기존 테이블 삭제
DROP TABLE IF EXISTS member_detail;

-- UNIQUE 제약조건을 추가하여 재생성
CREATE TABLE member_detail (
     member_detail_id BIGINT NOT NULL AUTO_INCREMENT, -- 회원 상세 ID (PK)
     member_id BIGINT NOT NULL, -- 회원 ID (FK, UNIQUE)
     bio TEXT NULL, -- 자기소개
     website_url VARCHAR(255) NULL, -- 개인 홈페이지
     mbti VARCHAR(10) NULL, -- MBTI

     PRIMARY KEY (member_detail_id),
     UNIQUE KEY uq_member_id (member_id), -- 이 부분이 핵심!
     CONSTRAINT fk_detail_member FOREIGN KEY (member_id)
     REFERENCES member (member_id)
);
```
   - UNIQUE KEY uq_member_id (member_id) : 이 한 줄이 member_id 컬럼의 모든 값이 고유해야 함을 보장
     + 즉, 특정 member_id 값을 가진 행은 이 테이블에 단 하나만 존재할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/6b098b16-209b-451f-bf72-2e0113f450ae">
</div>

   - 다시 데이터를 추가
```sql
-- '김개발' 회원의 첫 번째 상세 정보 추가 (성공)
INSERT INTO member_detail (member_id, bio, mbti)
VALUES (1, '안녕하세요, 최고의 개발자가 되고 싶습니다.', 'INTJ');
```

  - 이전에 문제가 되었던 두 번째 상세 정보 추가를 시도
```sql
-- '김개발' 회원의 두 번째 상세 정보 추가 (실패)
INSERT INTO member_detail (member_id, bio, mbti)
VALUES (1, '취미는 코딩입니다.', 'ENTP');
```

  - 실행 결과
```
Error Code: 1062. Duplicate entry '1' for key 'member_detail.uq_member_id'
```

   - 데이터베이스가 uq_member_id라는 UNIQUE 제약조건 위반으로 INSERT를 거부 : 이제 데이터 무결성이 완벽하게 지켜지는 1:1 관계가 완성
   - 이제 회원 목록 조회는 가벼워진 member 테이블만 읽으므로 매우 빠름
   - 그리고 상세 정보가 필요할 때만 다음과 같이 JOIN 을 사용하여 두 테이블을 연결하면 됨
```sql
-- 상세 정보가 필요할 때만 JOIN
SELECT m.name, d.bio, d.mbti
FROM member m
JOIN member_detail d ON m.member_id = d.member_id
WHERE m.member_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/196dad15-c390-46d2-88b0-536e0ea8487e">
</div>

6. 일대일 관계와 식별 관계
   - 일대일 관계를 구현하는 여러 방법 중에 부모 테이블의 PK를 자식 테이블의 PK + FK로 사용하는 방법도 존재
   - 이런 방식을 식별 관계라고 함
