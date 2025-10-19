-----
### 일대일(1:1) 관계 - 외래 키 위치
-----
1. 일대일(1:1) 관계에서는 양쪽 테이블 모두 '일(1)'의 위치에 있으므로, 이론적으로는 어느 쪽에 외래 키를 두어도 관계는 성립
   - 여기서는 자주 사용하는 핵심 테이블(member)을 주 테이블, 그리고 주 테이블의 부가 정보를 담는 테이블(member_detail)을 보조 테이블이라고 명명하고 두 가지 방법을 비교 분석
     + 방법 1 : 보조 테이블(member_detail)에 외래 키 (보조 테이블 FK → 주 테이블 PK 참조)
     + 방법 2 : 주 테이블(member)에 외래 키 (주 테이블 FK → 보조 테이블 PK 참조)
   - 결론부터 말하면, 실무에서는 특별한 이유가 없다면 방법 1을 권장

2. 방법 1 : 보조 테이블에 외래 키 두기 (권장)
   - 이 방법은 보조 테이블인 member_detail이 주 테이블인 member 의 기본 키(member_id)를 외래 키로 가지는 구조
   - 이 방식이 일대일 관계에서 가장 많이 사용하고, 또 기본으로 권장하는 방식
<div align="center">
<img src="https://github.com/user-attachments/assets/b5777d03-a834-4e84-9f71-69e5604ad439">
</div>

   - member_detail.member_id (FK) : 유니크 제약조건
   - 테이블 설계 및 예제 : member_detail의 member_id 컬럼에 외래 키(FK)와 유니크(UNIQUE) 제약조건을 함께 거는 것이 핵심
```sql
DROP TABLE IF EXISTS member_detail;
DROP TABLE IF EXISTS member;

-- 주 테이블
CREATE TABLE member (
     member_id BIGINT NOT NULL AUTO_INCREMENT,
     name VARCHAR(50) NOT NULL,
     PRIMARY KEY (member_id)
);

-- 보조 테이블
CREATE TABLE member_detail (
     member_detail_id BIGINT NOT NULL AUTO_INCREMENT, -- 고유 PK
     member_id BIGINT NOT NULL, -- FK + UNIQUE
     bio TEXT NULL,
     mbti VARCHAR(10) NULL,

     PRIMARY KEY (member_detail_id),
     UNIQUE KEY uq_member_id (member_id), -- 1:1 관계의 핵심
     CONSTRAINT fk_detail_member FOREIGN KEY (member_id)
     REFERENCES member (member_id)
);
```
   - 데이터 추가 및 조회 : 데이터를 추가하는 흐름은 매우 자연스러운데, 주 테이블인 member가 먼저 생성되고, 필요에 따라 member_detail이 추가
```sql
-- 1. 주 테이블에 데이터 추가
INSERT INTO member(name) VALUES ('김개발'); -- member_id = 1 가정

-- 2. 보조 테이블에 데이터 추가
INSERT INTO member_detail(member_id, bio, mbti)
VALUES (1, '항상 성장하는 개발자입니다.', 'INTJ');
```
   - 장점
     + 주 테이블의 독립성 및 확장성 : 이 설계의 가장 큰 장점
       * 주 테이블인 member는 member_detail의 존재 자체를 알지 못함
       * 따라서 나중에 seller (판매자), admin_auth (관리자 권한) 등 새로운 일대일 관계의 보조 테이블이 추가되어도 member 테이블의 구조는 전혀 변경할 필요가 없음
       * 각 보조 테이블이 member_id를 외래 키로 가지면 됨
       * 이는 시스템의 확장성을 매우 유연하게 만들어줌
     + 논리적 명확성 : '회원 상세 정보'는 '회원'에 종속되는 개념
       * 즉, 회원이 존재해야 상세 정보도 존재할 수 있음
       * 보조 테이블이 주 테이블을 참조하는 이 구조는 비즈니스 논리와 정확히 일치하여 모델을 이해하기 쉬움
     + 선택적 관계 표현의 용이성 : 모든 회원이 상세 정보를 가지지는 않으며, 이 구조에서는 상세 정보가 없는 회원은 그냥 member_detail 테이블에 행을 추가하지 않으면 그만임
       * 만약 반대로 member 테이블에 외래 키를 두었다면 선택적 관계를 위해 외래 키에 NULL 값을 입력해야 함
       * 결과적으로 member 테이블에는 NULL을 허용하는 불필요한 외래 키 컬럼이 추가되지 않아 깔끔함
     + 관계 확장의 유연성(1:1 → 1:N) : 비즈니스 요구사항은 계속 변함
       * "처음에는 하나만 가능했는데, 이제 여러 개가 필요해요"라는 요구는 매우 흔함
       * 이 설계는 이러한 변화에 매우 유연하게 대처할 수 있음
       * 예를 들어, 처음에는 게시글(board) 하나에 첨부파일(upload_file)을 하나만 등록할 수 있도록 설계했다고 가정 : 이후 유니크 제약조건만 제거하면 1:1 1:N으로 구조를 쉽게 변경할 수 있음
   - 단점
     + 관계 존재 여부 확인 : member 테이블에는 상세 정보에 대한 어떤 값도 없음
       * 따라서 특정 회원에게 상세 정보가 있는지 확인하려면 member_detail 테이블을 JOIN 하거나 EXISTS 서브쿼리를 사용해야 함

3. 참고 : 개방-폐쇄 원칙 (Open-Closed Principle, OCP)
   - 이 설계는 객체지향 설계 5원칙 중 하나인 OCP를 완벽하게 만족
   - 확장에는 열려있다 (Open for extension): seller, admin_auth 와 같은 새로운 기능을 얼마든지 추가(확장)할 수 있음
   - 수정에는 닫혀있다 (Closed for modification) : 새로운 기능이 추가되어도 핵심 테이블인 member의 코드는 수정할 필요가 없음
   - 좋은 설계는 이처럼 변화에 유연하게 대처할 수 있어야 함

4. 방법 2 : 주 테이블에 외래 키 두기
   - 이번에는 반대로 주 테이블인 member가 보조 테이블인 member_detail의 기본 키를 외래 키로 가지는 구조
<div align="center">
<img src="https://github.com/user-attachments/assets/c6718992-6c08-4f8f-9806-dec277dc46f9">
</div>

   - member.member_detail_id (FK) : 유니크 제약조건
   - 테이블 설계 및 예제
      + 💡 이 구조에서는 참조의 방향이 역전되므로, 테이블 생성 순서도 바뀜 : member_detail을 먼저 만들어야 이를 참조하는 member를 만들 수 있음
```sql
DROP TABLE IF EXISTS member_detail; -- member를 참조하므로, 먼저 제거
DROP TABLE IF EXISTS member;
```
   - 예제 마다 외래 키의 위치가 다르기 때문에 주의해야 함
```sql
DROP TABLE IF EXISTS member; -- member_detail을 참조하므로, 먼저 제거
DROP TABLE IF EXISTS member_detail;

-- 보조 테이블 (먼저 생성)
CREATE TABLE member_detail (
   member_detail_id BIGINT NOT NULL AUTO_INCREMENT,
   bio TEXT NULL,
   mbti VARCHAR(10) NULL,

   PRIMARY KEY (member_detail_id)
);

-- 주 테이블
CREATE TABLE member (
   member_id BIGINT NOT NULL AUTO_INCREMENT,
   member_detail_id BIGINT NULL, -- FK이자 UNIQUE, NULL 허용
   name VARCHAR(50) NOT NULL,

   PRIMARY KEY (member_id),
   UNIQUE KEY uq_member_detail_id (member_detail_id), -- 1:1 관계의 핵심
   CONSTRAINT fk_member_detail FOREIGN KEY (member_detail_id)
   REFERENCES member_detail (member_detail_id)
);
```
   - member_detail_id BIGINT NULL : 여기서 member 테이블의 member_detail_id 는 NULL 을 허용해야 함
     + 왜냐하면 상세 정보가 없는 회원도 존재할 수 있기 때문임
   - 물론 모든 회원이 상세 정보가 필수라면 NOT NULL 을 사용하면 됨
   - 데이터 추가 및 조회 : 데이터를 추가하는 과정이 부자연스럽고 복잡함
      + 주 테이블을 먼저 추가하는 경우
```sql
-- 1. 주 테이블에 데이터 추가 (보조 테이블 NULL)
INSERT INTO member(name, member_detail_id)
VALUES ('김개발', NULL);

-- 2. 보조 테이블에 먼저 데이터 추가
INSERT INTO member_detail(bio, mbti)
VALUES ('항상 성장하는 개발자입니다.', 'INTJ');

-- 3. 주 테이블에 보조 테이블 정보 등록
UPDATE member
SET member_detail_id = 1 -- 1이라고 가정
WHERE member_id = 1; -- 1이라고 가정
```

   - 주 테이블을 먼저 추가하는 경우 아직 보조 테이블이 없으므로 member_detail_id FK 값을 NULL로 입력
   - 이후에 보조 테이블이 추가되면 member_detail_id FK 값을 업데이트 
   - 데이터를 추가하는 과정이 복잡
   - 보조 테이블을 먼저 추가하는 경우
```sql
-- 1. 보조 테이블에 먼저 데이터 추가
INSERT INTO member_detail(bio, mbti)
VALUES ('기획은 나의 길, 기획자 입니다.', 'ENTJ');

-- 2. 주 테이블에 해당 ID를 넣어서 데이터 추가
INSERT INTO member(name, member_detail_id)
VALUES ('서기획', 2); -- member_detail_id: 2라고 가정
```
   - 주 테이블보다 보조 테이블의 데이터를 먼저 등록하므로 순서상 부자연스러움
   - 장점
     + 주 테이블 중심 설계 : 주 테이블이 모든 참조를 관리하므로, 핵심 엔티티(member)가 관계를 소유하는 느낌이 강함
       * 비즈니스 로직에서 주 테이블만 조회해도 보조 정보의 존재 여부를 바로 확인할 수 있음
     + 관계 존재 여부 확인의 용이성 : member 테이블이 member_detail_id 컬럼을 가지고 있음
       * 따라서 member 테이블의 member_detail_id 컬럼이 NULL 인지 아닌지만 확인하면 상세 정보의 유무를 즉시 알 수 있음
       * 즉, JOIN 이나 EXISTS 없이도 보조 테이블에 데이터가 있는지 확인할 수 있음
   - 단점
     + 나쁜 확장성 : 만약 seller 라는 새로운 일대일 관계가 추가되면 어떻게 해야 할까?
       * member 테이블에 seller_id라는 외래 키 컬럼을 또 추가해야 함
       * 새로운 일대일 관계가 생길 때마다 주 테이블의 구조를 변경(ALTER TABLE)해야 하는 끔찍한 상황에 부딪힘
       * member 테이블은 온갖 종류의 NULL 가능한 FK 컬럼들로 오염될 것
     + 관계 변경 유연성 : 위에서 설명한 1:1 → 1:N 관계로의 확장은 이 구조에서는 사실상 불가능하며, 테이블 구조를 완전히 재설계해야 함
     + NULL 값 허용 : member_detail_id는 모든 회원이 상세정보를 가지는 것은 아니므로 반드시 NULL을 허용해야 함
       * 이는 주 테이블에 불완전한 데이터가 들어갈 수 있음을 의미
       * 물론 모든 회원이 상세정보를 반드시 가져야 한다면 NOT NULL을 사용할 수 있음
     + 데이터 추가의 복잡성: 데이터 추가 로직이 상대적으로 복잡

5. 1:1 관계 - FK 위치에 따른 비교
<div align="center">
<img src="https://github.com/user-attachments/assets/102b9f4f-28c8-4044-861e-d3d7502b20ac">
</div>

   - 결론적으로, 일대일 관계를 설계할 때는 방법 1, 즉 보조 테이블에 외래 키를 두는 것이 좋음
