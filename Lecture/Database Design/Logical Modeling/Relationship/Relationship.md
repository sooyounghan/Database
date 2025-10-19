-----
### 논리적 모델링 - 관계
-----
1. 개념적 모델링에서 그렸던 추상적인 관계선이 논리적 모델링에서는 외래 키(Foreign Key)라는 구체적인 형태로 구현
   - 이 관계를 어떻게 설정하느냐에 따라 데이터 모델의 유연성과 확장성이 결정
2. 관계형 데이터베이스와 관계의 방향
   - 관계형 데이터베이스의 관계를 처음 배울 때 가장 흔하게 하는 오해 중 하나는 관계에 '방향'이 있다고 생각하는 것 : 특히 개발자의 경우 객체 지향 프로그래밍(OOP)의 참조 개념에 익숙하다 보니, 한쪽에서 다른 쪽으로만 접근할 수 있는 단방향 관계라고 생각하기 쉬움
   - 💡 결론부터 말하자면, 관계형 데이터베이스의 관계(Relationship)에는 방향이 없음 : 외래 키는 두 테이블을 연결하는 제약조건일 뿐, 데이터 조회 방향을 제한하지 않음
   - 예제 테이블 및 데이터 준비
      + 먼저 예제로 사용할 team과 member 테이블을 생성하고, 샘플 데이터를 추가
      + 하나의 팀(team)은 여러 회원(member)을 가질 수 있는 1:N 관계로 모델링
      + member 테이블의 team_id가 team 테이블의 team_id를 참조하는 외래 키가 됨
```sql
DROP TABLE IF EXISTS member_detail; -- 다른 예제에서 발생
DROP TABLE IF EXISTS member;
DROP TABLE IF EXISTS team;

-- 팀 테이블 생성
CREATE TABLE team (
   team_id BIGINT NOT NULL AUTO_INCREMENT,
   name VARCHAR(50) NOT NULL,
   PRIMARY KEY (team_id)
);

-- 회원 테이블 생성
CREATE TABLE member (
   member_id BIGINT NOT NULL AUTO_INCREMENT,
   team_id BIGINT NULL, -- 회원은 팀에 소속되지 않을 수도 있음 (NULL 허용)
   name VARCHAR(50) NOT NULL,

   PRIMARY KEY (member_id),
   CONSTRAINT fk_member_team FOREIGN KEY (team_id)
   REFERENCES team (team_id)
);
```
```sql
-- 샘플 데이터 추가
INSERT INTO team(name) VALUES ('개발팀'), ('기획팀');
INSERT INTO member(name, team_id) VALUES ('션', 1), ('잡스', 1), ('네이트', 2);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8c5eaf4d-713d-46eb-800a-923df9d97a3d">
</div>

3. 💡 객체 지향 프로그래밍의 방향성 있는 관계
   - Team과 Member 객체를 만든다고 가정
```java
class Team {
   Long teamId;
   String name;
   List<Member> members = new ArrayList<>(); // 팀에 소속된 회원 목록 참조
}

class Member {
   Long memberId;
   String name;
   // Member 객체는 어느 팀에 속했는지 모름 : Team 참조가 없음
}
```
   - Team Member 접근 O
      + 이 설계에서 Team 객체는 members 리스트를 통해 소속된 Member 객체들을 알고 있음
      + 따라서 team.members 처럼 팀에서 회원으로 접근하는 것은 매우 쉬움

   - Member Team 접근 X
     + 하지만 반대로, 특정 Member 객체만 보면 이 회원이 어떤 Team 에 속했는지 역으로 찾아가는 것은 불가능
      + Member 객체는 Team 에 대한 참조를 가지고 있지 않기 때문임

   - 이것이 바로 방향이 있는 단방향 관계
   - 만약 Member Team의 접근이 필요하다면 다음과 같이 Member -> Team 으로 접근할 수 있는 코드를 추가
```java
class Member {
   Long memberId;
   String name;
   Team team; // member -> team 접근 추가
}
```
   - 이렇게 서로 양쪽에서 참조할 수 있으면 양방향 관계가 됨

4. 💡 데이터베이스의 양방향 관계
   - 반면, 관계형 데이터베이스는 다름
   - member 테이블의 team_id 외래 키가 두 테이블을 연결하는 다리 역할을 하기 때문에, JOIN을 통해 어느 쪽에서든 자유롭게 양방향으로 데이터를 탐색할 수 있음
   - 회원(Member) 관점에서 팀(Team) 조회하기 : 먼저 외래 키가 있는 member를 중심으로 조회
     + '회원 잡스가 소속된 팀의 이름은 무엇인가?' 라는 요구사항을 처리 : member 테이블에서 시작
```sql
SELECT
   m.name AS member_name,
   t.name AS team_name
FROM member m
JOIN team t ON m.team_id = t.team_id
WHERE m.name = '잡스';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a593c57b-f551-4b0e-af8f-a3fb5dff2013">
</div>

   - 이 관계는 FK → PK로 조인

   - 팀(Team) 관점에서 회원(Member) 조회하기
     + 이번에는 외래 키가 없는 team 을 중심으로 조회
     + '개발팀 에 소속된 모든 회원의 이름을 조회하라'는 요구사항은 team 테이블에서 시작하여 member 테이블을 JOIN
```sql
SELECT
   t.name AS team_name,
   m.name AS member_name
FROM team t
JOIN member m ON t.team_id = m.team_id
WHERE t.name = '개발팀';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/0309660b-f9a5-4e5a-bce0-ac4ca9a9e22a">
</div>

   - 이 관계는 PK → FK로 조인

5. 💡 어떻게 FK 하나로 양방향 조회가 가능할까?
   - 조인은 FK → PK도 가능하지만, 그 반대인 PK → FK도 가능
   - 외래 키가 있는 곳에서는 조인을 사용해서 FK를 통해서 PK를 찾으면 되고, 그 반대편에서는 PK를 통해서 FK를 찾으면 됨
   - 이처럼 관계형 데이터베이스의 관계는 양방향 도로와 같음 : 객체 지향처럼 한쪽에서만 접근할 수 있는 일방통행 길이 아님
   - 💡 결과적으로 두 테이블이 관계를 맺으려면 FK를 둘 중 한 곳에 두면 됨 : 그럼 조인을 통해서 두 테이블을 서로 참조할 수 있음
   - 그럼 FK를 둘 중 어디에 두면 될까?
     + FK를 어디에 두는가에 따라 일대다와 같은 카디널리티를 포함한 수 많은 설계 정책에 영향을 줌

6. 관계의 2대 핵심 요소 : 카디널리티와 참여도
   - 논리적 모델링에서의 관계 역시 개념적 모델링에서와 마찬가지로 카디널리티(Cardinality)와 참여도(Optionality)라는 두 가지 핵심 속성을 가짐
   - 이 두 가지를 어떻게 조합하느냐에 따라 관계의 성격이 결정
      + 카디널리티 : 한 테이블의 행이 다른 테이블의 행과 몇 개나 연결될 수 있는지를 나타냄 (1:1, 1:N, N:1, M:N)
      + 참여도 : 한 테이블의 행이 관계를 맺고 있는 다른 테이블에 반드시 대응되는 행을 가져야 하는지(필수 참여), 아니면 갖지 않아도 되는지(선택 참여)를 나타냄
