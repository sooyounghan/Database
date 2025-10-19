-----
### 식별 관계 vs 비식별 관계 - 일대일(1:1)
-----
1. 회원(MEMBER)과 회원 상세정보(MEMBER_DETAIL)의 관계 예시 (1:1)
2. 비식별 관계 (Non-identifying) : '외래 키 위치'에서 살펴본 보조 테이블에 FK를 두는 방식이 바로 비식별 관계에 해당
<div align="center">
<img src="https://github.com/user-attachments/assets/6b052339-925d-476c-941a-2641b7326162">
</div>

   - 테이블 설계
```sql
DROP TABLE IF EXISTS member_detail_non_identifying;
DROP TABLE IF EXISTS member_non_identifying;

CREATE TABLE member_non_identifying (
     member_id BIGINT NOT NULL AUTO_INCREMENT,
     name VARCHAR(50) NOT NULL,
     PRIMARY KEY (member_id)
);

CREATE TABLE member_detail_non_identifying (
     member_detail_id BIGINT NOT NULL AUTO_INCREMENT, -- 독립적인 PK
     member_id BIGINT NOT NULL, -- 일반 컬럼 + FK + UNIQUE
     bio TEXT NULL,

     PRIMARY KEY (member_detail_id),
     UNIQUE KEY uq_member_id (member_id),
     CONSTRAINT fk_detail_member_non FOREIGN KEY (member_id)
     REFERENCES member_non_identifying (member_id)
);
```
   - member_detail_non_identifying 테이블은 자신만의 PK인 member_detail_id를 가짐
   - member_id는 member 테이블을 참조하는 FK 역할
   - uq_member_id : 1:1 관계 보장을 위해 UNIQUE 제약조건 추가

   - 데이터 입력 및 조회
```sql
-- 1. 회원 데이터 입력
INSERT INTO member_non_identifying (name)
VALUES ('kim'); -- member_id = 1 생성
INSERT INTO member_non_identifying (name)
VALUES ('park'); -- member_id = 2 생성

-- 2. 'kim' 회원의 상세 정보 입력
-- member_id = 1을 참조하여 상세 정보를 추가
INSERT INTO member_detail_non_identifying (member_id, bio)
VALUES (1, '데이터베이스 전문가입니다.'); -- member_detail_id = 1 생성
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8c73eac3-3230-4edf-b971-cf5d133c9970">
</div>

   - 이제 두 테이블을 JOIN 해서 회원과 상세정보를 함께 조회 : 상세정보가 없는 회원('park')도 함께 조회하기 위해 LEFT JOIN 
```sql
SELECT
   m.member_id,
   m.name,
   md.bio
FROM
   member_non_identifying m
LEFT JOIN
   member_detail_non_identifying md ON m.member_id = md.member_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/4e8e307d-22bf-4a65-8051-4915b5e5c9b6">
</div>

   - 결과를 보면, member_detail 테이블은 자신만의 member_detail_id를 PK로 가지며, FK인 member_id를 통해 member 테이블과 관계를 맺는 것을 확인할 수 있음
   - LEFT JOIN 을 통해 상세 정보가 없는 회원 정보도 누락 없이 조회할 수 있음

3. 식별 관계 (Identifying)
   - 이 방식에서는 자식 테이블(MEMBER_DETAIL)이 부모 테이블(MEMBER)의 PK를 그대로 물려받아 자신의 PK이자 FK로 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/757ee7ee-bec0-4c08-a239-d423b20efdd0">
</div>

   - 테이블 설계
```sql
DROP TABLE IF EXISTS member_detail_identifying;
DROP TABLE IF EXISTS member_identifying;

CREATE TABLE member_identifying (
   member_id BIGINT NOT NULL AUTO_INCREMENT,
   name VARCHAR(50) NOT NULL,
   PRIMARY KEY (member_id)
);
CREATE TABLE member_detail_identifying (
   member_id BIGINT NOT NULL, -- PK + FK
   bio TEXT NULL,

   PRIMARY KEY (member_id),
   CONSTRAINT fk_detail_member_iden FOREIGN KEY (member_id)
   REFERENCES member_identifying (member_id)
);
```

   - member_detail_identifying 테이블의 PK가 바로 member_id : 이 컬럼 하나가 PK와 FK 역할을 동시에 수행
   - 이 구조는 UNIQUE 제약조건 없이도 완벽한 1:1 관계를 보장 (PK는 본질적으로 UNIQUE와 NOT NULL)
   - 데이터 입력 및 조회
```sql
-- 1. 회원 데이터 입력
INSERT INTO member_identifying (name)
VALUES ('kim'); -- member_id = 1 생성
INSERT INTO member_identifying (name)
VALUES ('park'); -- member_id = 2 생성

-- 2. 'kim' 회원의 상세 정보 입력
-- member_id = 1을 PK이자 FK로 사용하여 상세 정보를 추가
INSERT INTO member_detail_identifying (member_id, bio)
VALUES (1, '데이터베이스 전문가입니다.');
```
<div align="center">
<img src="https://github.com/user-attachments/assets/d3990c60-7e55-4c14-b64f-5638480ec5d0">
</div>

   - 두 테이블을 JOIN 해서 회원과 상세정보를 함께 조회하는 쿼리는 비식별 관계일 때와 동일
```sql
SELECT
   m.member_id,
   m.name,
   md.bio
FROM
   member_identifying m
LEFT JOIN
   member_detail_identifying md ON m.member_id = md.member_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/745c8982-7538-4f43-8e83-af82b89caaed">
</div>

   - 결과를 보면 member_detail_identifying 테이블이 별도의 PK 없이 부모의 member_id를 그대로 PK이자 FK로 사용하는 것을 확인할 수 있음
   - 두 테이블의 member_id 가 완전히 동일한 값을 가지며 1:1로 매핑
   - 장단점 비교 및 실무 가이드 : 1:1 관계에서 식별 관계와 비식별 관계의 선택은 두 테이블의 논리적 관계를 어떻게 볼 것인지에 따라 결정
   - 비식별 관계 (Non-identifying)
      + 장점
        * 독립성과 유연성
          * member_detail 테이블이 member_detail_id라는 독립적인 PK를 가지므로, 두 테이블의 결합도(coupling)가 낮아짐
          * 이는 각 테이블을 더 독립적으로 관리하고 수정할 수 있게 해줌
          * 예를 들어, 나중에 member_detail 테이블이 다른 테이블과 새로운 관계를 맺어야 할 때, member_id가 아닌 member_detail_id를 FK로 제공하면 되므로 구조가 더 명확하고 유연

        * 일관성 있는 설계 : 대부분의 테이블은 독립적인 대리 키를 PK로 가지는 비식별 관계를 사용
          * 1:1 관계에서도 이 패턴을 일관되게 적용하면 전체 데이터베이스 스키마를 이해하고 유지보수하기 쉬워짐

      + 단점 : 1:1 관계를 보장하기 위해 별도의 UNIQUE 제약조건이 추가로 필요
   - 식별 관계 (Identifying)
      + 장점
        * 강한 논리적 결합 표현 : 두 테이블의 PK가 같기 때문에, member와 member_detail이 논리적으로는 하나의 개념이라는 점을 구조적으로 명확하게 표현할 수 있음
           * member_detail은 member 없이는 존재할 수 없다는 생명주기 의존성이 극명하게 드러남
        * 단순함 : PK가 FK를 겸하므로 UNIQUE 제약조건이 별도로 필요 없어 구조가 단순함
           * 또한 두 테이블을 조인할 때 항상 PK(member_id)끼리 조인하게 되므로 쿼리가 직관적

      + 단점
        * 낮은 유연성 : member_detail 테이블이 member 테이블에 강하게 종속 (만약 member_detail이 독립적으로 다른 테이블과 관계를 맺어야 하는 상황이 오면 설계가 복잡해질 수 있음)

4. 결론 및 실무 가이드
   - 특별한 경우가 아니라면 1:1 관계에서는 비식별 관계를 사용하는 것을 권장
   - 실무에서는 유연성과 확장성이 매우 중요한 가치 : 비식별 관계는 테이블 간의 결합도를 낮춰주어 향후 요구사항 변경에 더 유연하게 대처할 수 있도록 만들어줌
      + member와 member_detail처럼 논리적으로 매우 강하게 연결된 경우에도 비식별 관계를 사용하는 것이 더 나은 선택
   - 식별 관계는 두 테이블을 분리했지만, 논리적으로는 완벽하게 하나의 테이블로 취급해야 할 때 제한적으로 사용하는 것이 좋음
   - 예를 들어, 테이블의 컬럼이 너무 많아 자주 사용하는 컬럼 그룹과 거의 사용하지 않는 컬럼 그룹으로 나누어 성능을 최적화하는 경우에 식별 관계를 고려해볼 수 있음
   - 하지만 이런 경우를 제외하면, 비식별 관계가 더 안전하고 현명한 선택

5. 만약 1:1 관계를 일대다(1:N) 관계로 변경해야 한다면?
   - 소프트웨어 요구사항은 항상 변할 수 있음
   - 예를 들어, 처음에는 회원당 하나의 상세 정보만 허용했지만, 서비스가 발전하여 회원이 여러 개의 멀티 프로필(자기소개)을 가질 수 있도록 정책이 변경되었다고 가정
     + 이처럼 1:1 관계를 1:N으로 변경해야 할 때, 두 설계 방식은 극명한 차이를 보임

   - 비식별 관계의 경우 - 유연한 대처
     + 비식별 관계에서 1:1을 보장하는 핵심은 member_detail_non_identifying 테이블의 member_id 컬럼에 걸려있는 UNIQUE 제약조건
     + 따라서 1:N 관계로 변경하는 작업은 이 제약조건을 삭제하기만 하면 끝남
     + 이처럼 테이블의 기본 구조나 PK는 전혀 건드리지 않고, 제약조건 하나만 변경하여 손쉽게 요구사항 변화에 대응할 수 있음
     + 이것이 바로 비식별 관계가 가진 유연성과 확장성

   - 식별 관계의 경우 - 대규모 공사
     + 식별 관계에서 1:1을 보장하는 핵심은 member_detail_identifying 테이블의 기본 키(PK)가 member_id라는 사실 그 자체
     + PK는 본질적으로 중복될 수 없으므로 1:1 관계가 강제
     + 이것을 1:N 관계로 바꾸려면 PK 자체를 바꿔야 함
     + 이는 테이블의 근간을 바꾸는 대규모 작업

   - 변경 절차
     + member_detail_identifying 테이블에 member_detail_id 같은 새로운 PK 컬럼을 추가
     + 기존의 PK였던 member_id는 더 이상 PK가 아니게 되므로, PK 제약조건을 삭제
     + 새로 추가한 member_detail_id를 새로운 PK로 지정
     + member_id는 이제 일반적인 FK 역할만 수행

6. 결국, 식별 관계 모델을 포기하고 비식별 관계 모델로 완전히 전환하는 것과 같은 작업이 필요
   - 이는 매우 복잡하고 위험하며, 이 테이블을 참조하는 모든 코드를 수정해야 할 수도 있음
   - 이것이 바로 식별 관계가 변경에 취약하고 경직된 구조라고 불리는 이유
   - 결론적으로, 1:1 관계에서는 비식별 관계를 사용하는 것이 더 유연하고 확장성 있는 좋은 설계
