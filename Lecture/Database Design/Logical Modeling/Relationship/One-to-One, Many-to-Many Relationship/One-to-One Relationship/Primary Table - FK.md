-----
### 주 테이블에 FK
-----
1. 대부분의 경우 보조 테이블에 FK를 두는 방법이 정답에 가깝지만, 모든 설계에 하나의 정답만 있는 것은 아님
2. 경우 1 : 관계가 선택이 아닌 필수일 때
   - 만약 비즈니스 요구사항이 '모든 회원은 반드시 상세 정보를 가져야 한다'고 강제하는 경우를 가정
   - 즉, member와 member_detail은 생성과 소멸의 생명주기(Lifecycle)를 완전히 함께하는, 사실상 하나의 논리적 덩어리
   - 보조 테이블에 FK를 두는 방식을 사용하면 상세 정보가 없는 회원도 존재할 수 있음
```
1. member 저장
2. member_detail 저장하지 않음
```
   - 여기서 2와 같이 member_detail을 저장하는 것을 실수로 깜박하면 member만 추가될 수 도 있음
   - 따라서 보조 테이블에 FK를 두는 방식은 모든 회원은 반드시 상세 정보를 가져야 한다는 조건을 데이터베이스 관계로 강제할 수 없음
   - 대신에 주 테이블에 FK를 두는 방식을 사용하면 어떻게 될까?
      + 주 테이블인 member에 member_detail_id를 필수(NOT NULL)로 두는 설계를 고려할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/3eb6a82e-243e-45fe-aaf3-e1ee5f995667">
</div>

   - 테이블 설계 예제
```sql
DROP TABLE IF EXISTS member;
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
   member_detail_id BIGINT NOT NULL, -- NOT NULL 제약조건이 핵심
   name VARCHAR(50) NOT NULL,

   PRIMARY KEY (member_id),
   UNIQUE KEY uq_member_detail_id (member_detail_id),
   CONSTRAINT fk_member_detail FOREIGN KEY (member_detail_id)
   REFERENCES member_detail (member_detail_id)
);
```
   - 데이터 정합성 보장 : member_detail_id가 NOT NULL 이므로 데이터베이스 수준에서 회원이 상세 정보 없이 생성되는 것을 원천적으로 차단하며, 데이터의 무결성을 강력하게 보장할 수 있음
   - NULL 컬럼 방지 : 주 테이블의 단점이었던 NULL 허용 문제가 사라짐
   - 데이터 추가 및 제약조건 확인
     + 이 설계에서는 데이터를 추가하는 순서가 매우 중요함 : member 테이블이 member_detail의 member_detail_id를 참조하므로, 반드시 member_detail 행을 먼저 생성해야 함
     + 만약 이 순서를 지키지 않거나 member_detail 없이 member 를 만들려고 하면 어떻게 될까?

3. 제약조건 위반 시나리오 1 : 상세 정보 없는 회원을 추가
   - member 테이블의 member_detail_id는 NOT NULL 제약조건을 가지고 있음
   - 만약 이 컬럼에 NULL 값을 넣어서 새로운 회원을 추가하려고 시도하면 어떻게 될까?
```sql
INSERT INTO member (member_detail_id, name)
VALUES (NULL, '네이트');
```
  - 실행 결과
```
Error Code: 1048. Column 'member_detail_id' cannot be null
```
   - 예상대로 데이터베이스는 NOT NULL 제약조건 위반 오류를 발생시키며 데이터 추가를 거부
   - 이처럼 주 테이블에 FK를 두고 NOT NULL 제약조건을 거는 방식은 모든 회원이 상세 정보를 반드시 가지도록 데이터베이스 수준에서 강력하게 강제

4. 제약조건 위반 시나리오 2 : member_detail에 존재하지 않는 member_detail_id 값(예) 100)으로 member를 추가
```sql
-- 존재하지 않는 member_detail_id(100)를 사용
INSERT INTO member (member_detail_id, name)
VALUES (100, '네이트');
```
  - 실행 결과
```
Error Code: 1452. Cannot add or update a child row: a foreign key constraint
fails (`my_shop3`.`member`, CONSTRAINT `fk_member_detail` FOREIGN KEY
(`member_detail_id`) REFERENCES `member_detail` (`member_detail_id`))
```
   - FOREIGN KEY 제약조건 위반 오류가 발생하며 데이터 추가가 실패
   - 이처럼 데이터베이스는 존재하지 않는 상세 정보를 참조하려는 잘못된 데이터의 생성을 원천적으로 막아줌

5. 올바른 데이터 추가 순서
   - 데이터를 올바르게 추가하는 순서
      + member_detail에 먼저 데이터를 추가
      + 생성된 member_detail_id를 확인
      + 확인된 member_detail_id를 사용해서 member 테이블에 데이터를 추가
   - 회원 '네이트'와 '션'의 데이터를 추가
```sql
-- 1. '네이트'의 상세 정보 추가
INSERT INTO member_detail (bio, mbti)
VALUES ('안녕하세요. 백엔드 개발자입니다.', 'INFJ');

-- 2. '네이트'의 회원 정보 추가 (방금 생성된 member_detail_id는 1)
INSERT INTO member (member_detail_id, name)
VALUES (1, '네이트');

-- 3. '션'의 상세 정보 추가
INSERT INTO member_detail (bio, mbti)
VALUES ('풀스택 개발자를 꿈꿉니다.', 'ENFP');

-- 4. '션'의 회원 정보 추가 (방금 생성된 member_detail_id는 2)
INSERT INTO member (member_detail_id, name)
VALUES (2, '션');
```
   - 데이터 조회
     + 이제 JOIN을 사용해서 회원과 상세 정보를 함께 조회
     + NOT NULL 제약조건 덕분에 모든 회원은 반드시 상세 정보를 가지므로, INNER JOIN 을 사용해도 데이터가 누락될 걱정이 없음
```sql
SELECT
   m.member_id,
   m.name,
   md.bio,
   md.mbti
FROM
   member m
JOIN
   member_detail md ON m.member_detail_id = md.member_detail_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/4b963dfe-5618-461d-8801-46b5b8c9ff9a">
</div>

6. 외래 키(FK) 위치에 따른 데이터 정합성 비교 - 정리
   - 데이터베이스를 설계할 때 일대일(1:1) 관계에서 외래 키(FK)를 주 테이블에 둘지, 보조 테이블에 둘지에 따라 데이터의 정합성을 보장하는 수준이 달라짐
   - 보조 테이블에 FK를 두는 방식
     + 이 방식은 관계가 선택적일 때 유용
     + 예를 들어, 모든 회원이 상세 정보를 필수로 가져야 하는 것이 아니라면 이 방법이 더 유연
     + 한계 : 회원 정보 member 테이블)는 생성되었지만, 상세 정보(member_detail 테이블)는 누락되는 상황을 데이터베이스 수준에서 막을 수 없으며, 애플리케이션 코드에서 이 부분을 따로 처리해야 함

   - 주 테이블에 FK를 두는 방식 (NOT NULL 제약조건 포함)
     + 이 방식은 두 테이블의 관계가 필수적일 때 데이터 무결성을 강력하게 보장
     + 강점 : 주 테이블(member)에 있는 외래 키(member_detail_id)에 NOT NULL 제약조건을 설정하면, 반드시 보조 테이블(member_detail)에 해당 데이터가 먼저 존재해야만 주 테이블에 데이터를 추가할 수 있음
     + 결과 : '상세 정보가 없는 회원'이 생성되는 것을 원천적으로 차단하여 데이터의 정합성을 완벽하게 유지할 수 있음

7. 실무 팁
   - 이러한 설계는 두 테이블이 논리적으로 너무 강하게 묶여 있어 사실상 하나의 테이블처럼 취급될 때 사용을 고려해 볼 수 있음
   - 하지만 대부분의 경우, 미래의 확장성을 고려하면 관계가 필수인 것처럼 보여도 '보조 테이블에 FK를 두는 방법'으로 설계하고 애플리케이션 레벨에서 데이터 존재 여부를 검증하는 것이 더 유연한 해결책일 수 있음

8. 경우 2 : 조회 성능이 극도로 중요할 때
   - 드문 경우지만, 주 테이블(member)을 조회할 때 보조 테이블(member_detail)의 존재 여부를 확인하는 로직이 시스템의 성능에 심각한 병목을 일으키는 상황을 가정
     + 예를 들어, 초당 수천 건의 회원 목록 조회가 발생하는데, 각 회원마다 상세 프로필이 있는지 없는지를 아이콘으로 표시해야 한다고 가정
   - 보조 테이블에 FK가 있는 경우
     + 회원의 수만큼 JOIN 이나 EXISTS 서브쿼리를 실행해야 하므로, 데이터 양이 많아지면 성능 저하가 발생할 수 있음
 ```sql
 -- 회원 목록과 함께 상세 정보 존재 여부를 확인 (JOIN 사용)
SELECT m.member_id, m.name,
       CASE WHEN md.member_id IS NOT NULL THEN 'Y' ELSE 'N' END AS has_detail
FROM member m
LEFT JOIN member_detail md ON m.member_id = md.member_id;
 ```
  - 주 테이블에 FK가 있는 경우
    + JOIN 없이 member 테이블만 조회하면 되므로 훨씬 빠름
 ```sql
-- 회원 목록과 함께 상세 정보 존재 여부를 확인 (단일 테이블 조회)
SELECT member_id, name,
       CASE WHEN member_detail_id IS NOT NULL THEN 'Y' ELSE 'N' END AS has_detail
FROM member;
```
   - 장점
     + 조회 성능 향상 : JOIN을 피할 수 있어 특정 조회 시나리오에서 성능 이점을 가질 수 있음
   - 단점
     + 설계 유연성 저하 : 성능 하나를 위해 확장성, 유연성, 데이터 추가의 단순함 등 많은 것을 희생해야 함
  
9. 섣부른 최적화는 금물
   - 대부분의 애플리케이션에서는 '방법 1'의 JOIN 성능이 문제가 되지 않음
   - 인덱스 설계를 잘하는 것만으로도 충분히 빠름
   - 성능 테스트를 통해 명확한 병목 지점이 확인되었을 때, 다른 모든 튜닝 방법을 시도한 후에도 해결되지 않을 때 비로소 고려해 볼 만한 전략
   - 처음부터 성능을 예측하여 이 방법을 선택하는 것은 나쁜 설계로 이어질 가능성이 높음
   - 결론적으로, 보조 테이블에 FK를 두는 방법이 더 나은 선택이지만, 위와 같은 극히 예외적인 상황과 그에 따른 장단점을 이해하고 있다면 더 넓은 시야를 가진 데이터베이스 설계자가 될 수 있을 것

-----
### 주 테이블에 외래 키를 고려할 수 있는 다양한 경우들
-----
1. 일대일 관계에서 주 테이블에 외래 키를 두는 것을 고려할 수 있는 다양한 경우를 정리
   - 보조 테이블이 주 테이블보다 먼저 생성, 관리되는 경우
     + 예) 사전에 profile 데이터가 생성되고, 이후 해당 프로필을 참조하는 user가 만들어지는 서비스
     + 데이터 등록 순서가 "보조 → 주"로 고정되어 있고, 보조 데이터가 독립적으로 유효성을 가져야 하는 경우 주 테이블에 FK를 두면 등록 로직이 단순해질 수 있음
   - 보조 테이블이 주 테이블의 필수 구성 요소인 경우
     + 회원 가입 시 반드시 상세 정보가 같이 생성되어야 하는 경우
     + 모든 주 테이블이 보조 테이블을 반드시 가져야 하는 구조에서는 외래 키(FK)를 NOT NULL로 설정할 수 있으므로, 주 테이블에 FK를 두어도 NULL 값 허용으로 인한 불완전한 데이터 문제가 발생하지 않음
     + 예) passport와 person처럼, 한 사람이 반드시 여권을 가져야 하는 경우
   - 조회 빈도가 매우 높고, JOIN 비용을 최소화해야 하는 경우
     + 주 테이블만 조회해도 보조 정보의 존재 여부나 식별자를 바로 알아야 하는 경우
     + 특히 API 응답에서 주 테이블 데이터와 보조 테이블의 ID를 함께 내려주는 일이 많다면 FK를 주 테이블에 두는 것이 효율적일 수 있음
   - ORM 매핑 편의성
     + 일부 ORM에서는 주 테이블이 FK를 가지면 양방향 매핑 없이도 보조 객체를 직접 로딩하기 쉬운 경우가 있음
     + 예) JPA에서 @OneToOne 관계를 주 테이블이 FK로 가지면 fetch 설정과 연관관계 주인이 단순해질 수 있음

2. 일대일 관계의 외래 키 - 결론
   - 일대일 관계에서 외래 키(FK)의 위치는 시스템의 확장성과 유연성에 큰 영향을 미침
   - 일대일 관계에서는 보조 테이블에 외래 키를 두는 것을 표준으로 삼아야 함
   - 핵심 원칙
      + 기본 선택 (권장) : 보조 테이블에 외래 키(FK) 설정
         * 주 테이블의 변경 없이 새로운 관계를 추가하거나, 1:N 관계로 전환하기 쉬움
         * 이는 확장성과 유연성을 보장하며 개방-폐쇄 원칙(OCP)을 따르는 좋은 설계로, 대부분의 상황에서 이 방법을 선택해야 함
      + 예외적 선택: 주 테이블에 외래 키(FK) 설정
         * 이 방법은 확장성을 포기하는 대신 명확한 이점을 얻고자 할 때만 사용해야 함
         * 두 테이블의 관계가 필수적일 때 (NOT NULL 제약으로 데이터 정합성 강제)
         * JOIN을 피해야 할 만큼 조회 성능이 매우 중요할 때
         * 특정 ORM 매핑의 편의성이 우선될 때
   - 좋은 데이터베이스 설계는 현재의 요구사항을 만족시키는 것을 넘어, 미래의 변화를 얼마나 유연하게 수용할 수 있는가에 달려있음
   - 이런 관점에서 보조 테이블에 외래 키를 두는 것을 기본으로 선택
