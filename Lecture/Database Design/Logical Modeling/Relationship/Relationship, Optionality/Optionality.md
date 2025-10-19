-----
### 참여도
-----
1. 관계의 핵심 요소로는 카디널리티와 함께 참여도(Optionality)가 있음
   - 참여도는 한 엔티티가 관계에 필수적으로 참여해야 하는지(Mandatory), 아니면 선택적으로 참여할 수 있는지(Optional)를 나타냄
   - 필수적 관계(Mandatory Relationship) : 관계를 맺는 상대방 엔티티에 반드시 대응되는 행이 있어야 함 (예) 모든 회원은 반드시 팀에 소속되어야 함)
   - 선택적 관계(Optional Relationship) : 상대방 엔티티에 대응되는 행이 없어도 됨 (예) 회원은 팀에 소속될 수도 있고, 아닐 수도 있음)

2. 논리적 모델링에서 이 참여도는 외래 키(FK) 컬럼의 NULL 허용 여부(NULL 또는 NOT NULL)로 구현 : 하지만 이 제약조건만으로 모든 비즈니스 규칙을 강제할 수는 없음
     
3. 외래 키와 참여도 구현
   - 팀(team)과 회원(member)의 예
   - 외래 키가 있는 '다(N)' 쪽, 즉 member 테이블에서 참여도를 구현하는 것은 매우 직관적
   - 회원은 팀에 소속될 수 있음 (선택적 참여)
     + 비즈니스 규칙
       * 회원은 하나의 팀에 소속될 수도 있고, 아직 소속되지 않은 상태일 수도 있음
       * 팀은 여러 회원을 가질 수 있음 :  팀은 회원을 필수로 가져야 함
     + 구현 : member 테이블의 team_id 외래 키 컬럼을 NULL 허용으로 설정
<div align="center">
<img src="https://github.com/user-attachments/assets/62c85ea9-99cb-47c2-874b-9e41e7178b76">
</div>

   - team_id FK 빨간색 마름모가 비어있는 것을 확인할 수 있음 : NULL 허용이라는 뜻
   - member → team 이 O로 선택적 관계인 것을 확인할 수 있음
   - 까마귀발 표기법 - Crow’s Foot
      + 까마귀발 표기법을 읽을 때 가장 중요한 원칙은 "한 엔티티 쪽 끝에 있는 기호는 반대편 엔티티에 대한 규칙을 설명"하는 것
      + 관계선 끝의 기호는 반대편 엔티티의 최소, 최대 참여 수(필수 / 선택, 1 / 여러)를 뜻
```sql
DROP TABLE IF EXISTS member;
DROP TABLE IF EXISTS team;

CREATE TABLE team (
   team_id BIGINT NOT NULL AUTO_INCREMENT,
   name VARCHAR(50) NOT NULL,
   PRIMARY KEY (team_id)
);

CREATE TABLE member (
   member_id BIGINT NOT NULL AUTO_INCREMENT,
   team_id BIGINT NULL, -- NULL 허용으로 선택적 참여 구현
   name VARCHAR(50) NOT NULL,

   PRIMARY KEY (member_id),
   CONSTRAINT fk_member_team FOREIGN KEY (team_id)
   REFERENCES team (team_id)
);
```
  - 팀이 정해지지 않은 신입 회원을 team_id 없이도 member 테이블에 추가할 수 있음

```sql
INSERT INTO member(name, team_id) VALUES ('이기획', NULL);
```
   - team_id 가 NULL 이므로 성공적으로 실행
```sql
SELECT *
FROM member;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/1481aa51-7006-4bcc-b0c1-030056fe932d">
</div>

   - 회원은 반드시 팀에 소속되어야 함 (필수적 참여)
     + 비즈니스 규칙 : 모든 회원은 가입과 동시에 반드시 특정 팀에 소속되어야 함    
     + 구현 : member 테이블의 team_id 외래 키 컬럼을 NOT NULL로 설정
<div align="center">
<img src="https://github.com/user-attachments/assets/8cafe73c-7c95-441c-8648-52aff445761c">
</div>

   - team_id FK 빨간색 마름모가 꽉 차있는 것을 확인할 수 있음 : NOT NULL 이라는 뜻
   - member → team이 필수적 참여이므로 O → ```|``` 로 변한 것을 확인할 수 있음
```sql
DROP TABLE IF EXISTS member;

CREATE TABLE member (
 member_id BIGINT NOT NULL AUTO_INCREMENT,
 team_id BIGINT NOT NULL, -- NOT NULL로 필수적 참여 구현
 name VARCHAR(50) NOT NULL,

 PRIMARY KEY (member_id),
 CONSTRAINT fk_member_team FOREIGN KEY (team_id)
 REFERENCES team (team_id)
);
```
   - 이제 팀 정보 없이 회원을 추가하려고 하면 데이터베이스가 오류를 발생시켜 참조 무결성을 지켜줌
```sql
INSERT INTO member(name, team_id) VALUES ('이기획', NULL);
```
  - 실행 결과
```
Error Code: 1048. Column 'team_id' cannot be null
```
   - 이처럼 외래 키가 있는 쪽의 참여도는 NULL 제약조건으로 간단하고 강력하게 제어할 수 있음

4. 관계 반대편의 참여도 : 데이터베이스 제약의 한계
   - 그렇다면 관계의 반대편, 즉 '일(1)' 쪽인 team 테이블 입장에서의 참여도는 어떨까?
   - 선택적 참여 : 팀은 회원이 한 명도 없을 수 있음 (예) 신설팀)
<div align="center">
<img src="https://github.com/user-attachments/assets/d2f57453-1d81-48f2-b72a-3abb56609d04">
</div>

   - team → member이 선택적 참여이므로 O
     
   - 필수적 참여 : 팀은 생성될 때 반드시 최소 한 명 이상의 회원을 가져야 함
<div align="center">
<img src="https://github.com/user-attachments/assets/1783bdda-51c1-471b-817b-d122805909ab">
</div>

   - team → member가 필수적 참여이므로 O  → ```|```로 변한 것을 확인할 수 있음

   - 선택적 참여는 데이터베이스에서 자연스럽게 구현
     + 그냥 team 테이블에 새로운 팀을 추가하기만 하면 됨
     + member 테이블에 해당 팀을 참조하는 데이터가 있든 없든 상관없음
     + 이 관계에서 team 테이블에만 데이터를 넣는다고 생각 : member 없이 team에만 데이터를 넣어도 아무런 문제가 없음

   - 하지만 필수적 참여, 즉 '팀은 최소 한 명 이상의 회원을 가져야 한다'는 규칙은 어떻게 강제할 수 있을까?
     + 결론부터 말하면, 외래 키와 같은 기본 제약조건만으로는 이 규칙을 강제할 수 없음

   - 💡 왜 데이터베이스 제약으로 강제할 수 없을까?
     + 이유는 바로 제약을 걸 대상, 즉 외래 키 컬럼이 team 테이블에 존재하지 않기 때문임
     + member 테이블에서는 team_id 라는 명확한 외래 키 컬럼이 있었음 : 이 컬럼에 NOT NULL 제약조건을 걸어서 '팀 없는 회원'의 생성을 막을 수 있었음
     + 즉, 규칙을 강제할 수단(컬럼)이 명확히 있었음
     + 하지만 team 테이블에는 회원을 가리키는 어떠한 외래 키도 존재하지 않으므로, 따라서 NOT NULL과 같은 제약조건을 걸 대상 자체가 없는 것
     + 데이터베이스 입장에서는 규칙을 강제할 수단이 구조적으로 없는 셈

5. 💡 ERD를 비교
   - member → team의 관계는 member에 있는 team_id FK에 NULL, NOT NULL 제약조건을 사용해서 필수적 선택적 관계를 변경
     + team_id FK 마름모가 비어있고, 꽉차있는 모습으로 변함
   - 반면에 team → member 의 관계는 team에 FK가 없음
     +  따라서 team -> member 의 관계를 제약할 수 있는 방법이 없음
   - 결국 관계형 데이터베이스의 제약조건 만으로는 외래 키가 없는 team → member와 같은 관계는 필수적 관계로 만들 수 없음

6. 이러한 종류의 비즈니스 규칙은 보통 애플리케이션 계층(Application Layer)에서 로직으로 해결
   - 예를 들면 애플리케이션 계층에서 다음과 같은 쿼리를 하나의 트랜잭션으로 수행하는 것
```sql
-- 팀 생성과 첫 멤버 등록을 하나의 트랜잭션으로 처리
START TRANSACTION;

INSERT INTO team(name) VALUES ('플랫폼팀');
SET @team_id := LAST_INSERT_ID();

INSERT INTO member(name, team_id) VALUES ('션', @team_id);

COMMIT;
```

7. 데이터베이스 고급 기능
   - 데이터베이스에 따라 트리거(Trigger)나 같은 기능을 사용해 이런 복잡한 규칙을 구현할 수도 있음
   - 하지만 이는 데이터베이스에 비즈니스 로직이 깊게 관여하게 만들어 애플리케이션과의 의존성을 높이고 관리를 어렵게 할 수 있으므로, 신중하게 사용해야 함
   - 대부분의 현대적인 개발 방식에서는 애플리케이션에서 트랜잭션으로 처리하는 것을 더 선호

8. ERD와 비즈니스 규칙의 중요성
   - 데이터베이스 제약조건으로 구현하기 어렵다고 해서, ERD(개체관계 다이어그램)에 원래의 비즈니스 규칙을 표현하지 않으면 안 된다는 것
     + ERD : 데이터베이스의 물리적 제약을 넘어, 원래의 비즈니스 규칙을 그대로 표현해야 함 - 만약 '팀은 반드시 한 명 이상의 회원을 가져야 한다'는 규칙이 있다면, ERD에는 그 필수 참여 관계가 명확하게 그려져야 함
     + 데이터베이스 DDL : ERD에 표현된 규칙 중, 데이터베이스가 제약조건으로 강제할 수 있는 부분까지만 구현
     + 애플리케이션 코드 : DDL로 구현하지 못한 나머지 비즈니스 규칙을 책임지고 구현하여 데이터의 무결성을 완성

   - 예를 들어, 다음과 같은 비즈니스 규칙이 있다고 가정
    + 모든 회원은 반드시 하나의 팀에 소속되어야 함 (필수 참여)
    + 팀은 생성될 때 반드시 최소 한 명 이상의 회원을 가져야 함 (필수 참여)
<div align="center">
<img src="https://github.com/user-attachments/assets/03781219-67ee-4014-8f71-a724b6b960dc">
</div>

   - team 쪽의 || : member는 team 에 반드시 참여해야 함 (필수 참여)
   - member 쪽의 |< : team은 member 를 반드시 가져야 함 (필수 참여)

   - DDL 구현
     + member.team_id 는 NOT NULL로 설정 (규칙 1 구현)
     + 💡 team 테이블에는 FK가 없으므로 제약조건으로 강제할 수 없음 : 따라서 애플리케이션에서 구현해야 한다. (규칙 2 구현)

9. 결론적으로, 데이터베이스 설계는 단순히 테이블과 컬럼을 만드는 작업이 아님
    - 비즈니스 규칙을 명확히 이해하고, 그규칙을 ERD에 올바르게 표현한 뒤, 데이터베이스 제약조건과 애플리케이션 코드가 각자의 역할을 분담하여 전체 데이터의 무결성을 지켜나가는 과정
