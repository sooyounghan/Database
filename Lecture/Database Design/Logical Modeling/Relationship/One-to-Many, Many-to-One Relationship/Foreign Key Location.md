-----
### 일대다(1:N) 다대일(N:1) 관계 - 외래 키 위치
-----
1. 일대다(1:N) 다대일(N:1) 관계는 가장 흔하게 볼 수 있는 관계
   - 실무에서 만나는 대부분의 관계는 일대다 또는 다대일 관계라고 봐도 무방
   - 팀(Team) 과 회원(Member) 에시
<div align="center">
<img src="https://github.com/user-attachments/assets/34559403-640f-4e76-b879-3c4cc98a8015">
</div>

   - 하나의 팀에는 여러 명의 회원이 소속될 수 있음
   - 한 명의 회원은 반드시 하나의 팀에만 소속됨
   - 여기서는 둘 다 선택적 관계를 사용

2. 이 관계를 어느 테이블 입장에서 바라보느냐에 따라 관계의 카디널리티가 달라짐
   - 팀 입장에서 회원 을 바라보면, 하나의 팀이 여러 회원을 가지므로 일대다(1:N) 관계
   - 회원 입장에서 팀 을 바라보면, 여러 회원이 하나의 팀에 속하므로 다대일(N:1) 관계
   - 결국 같은 관계를 어느 쪽에서 바라보느냐의 차이일 뿐임
   - 논리적 모델링에서 중요한 점은 이 관계를 데이터베이스 테이블로 구현하는 방법

3. 외래 키(Foreign Key)는 어디에 위치해야 할까?
   - 관계형 데이터베이스는 외래 키를 사용해서 관계를 표현
   - 💡 그렇다면 일대다 또는 그 반대인 다대일 관계에서 외래 키는 어디에 위치해야 할까?
      + 결론부터 말하면, 외래 키는 항상 '다(N)' 쪽에 존재해야 함
      + 즉, 회원(N) 테이블이 팀(1) 테이블의 기본 키를 외래 키로 가져야 함
   - '일(1)' 쪽에 외래 키를 두면 발생하는 문제 1 : 여러 행에 나누어 보관
     + 하나의 팀은 하나의 행에 저장해야 함
     + 따라서 다음과 같이 여러 행을 저장하는 것은 말이 안 됨
```sql
DROP TABLE IF EXISTS team_bad1;

CREATE TABLE team_bad1 (
   team_id BIGINT NOT NULL,
   name VARCHAR(50) NOT NULL,
   member_id BIGINT NULL,
   PRIMARY KEY (team_id)
);
```
```sql
-- 데이터 삽입
INSERT INTO team_bad1(team_id, name, member_id) VALUES (1, '개발팀', 1); -- 김개발 (ID:1)
-- 오류 발생
INSERT INTO team_bad1(team_id, name, member_id) VALUES (1, '개발팀', 2); -- 박개발 (ID:2)
```
   - 기대하는 결과
<div align="center">
<img src="https://github.com/user-attachments/assets/0edeb6b7-ae39-4a3f-a3f1-8622cfc89fa5">
</div>

   - 실행 결과
```
Error Code: 1062. Duplicate entry '1' for key 'team_bad1.PRIMARY'
```
   - 팀 테이블에 같은 팀을 여러 행 저장해서는 안 됨
   - 두 번째 회원을 저장하는 순간 오류가 발생
   - '일(1)' 쪽에 외래 키를 두면 발생하는 문제 2 : 하나의 컬럼에 보관
      + 그렇다면 하나의 컬럼 안에 여러 member_id를 보관할 경우 : 예를 들어 member_ids 라는 컬럼 하나에 '1,2'와 같은 문자열을 넣는 방식은 어떨까?
```sql
DROP TABLE IF EXISTS team_bad2;

CREATE TABLE team_bad2 (
   team_id BIGINT NOT NULL,
   name VARCHAR(50) NOT NULL,
   member_ids VARCHAR(255) NULL, -- 쉼표로 구분된 회원 ID 목록
  
   PRIMARY KEY (team_id)
);
```
```sql
-- 데이터 삽입
INSERT INTO team_bad2(team_id, name, member_ids) VALUES (1, '개발팀', '1,2');
```
   - 이 방식은 관계형 데이터베이스의 가장 기본 원칙을 위배하는, 매우 심각한 문제를 가진 설계
      + 제1정규형(1NF) 위반 : 관계형 데이터베이스의 컬럼은 원자성(Atomicity)을 가져야 함
        * 즉, 컬럼 하나에는 단 하나의 값만 저장되어야 한다는 원칙
        * '1, 2'처럼 여러 값을 하나의 문자열로 욱여넣는 것은 이 원칙을 정면으로 위배
      + 데이터 검색의 어려움 : '박개발(ID:2)' 회원이 소속된 팀을 찾아라 라는 간단한 요구사항을 처리하기가 매우 까다로움
```sql
SELECT name
FROM team_bad2
WHERE member_ids LIKE '%2%';
```
   - 이런 쿼리는 인덱스를 전혀 활용할 수 없어 테이블의 모든 데이터를 하나씩 다 확인하는 '풀 테이블 스캔(Full Table Scan)' 방식으로 동작 : 데이터가 많아질수록 성능은 끔찍하게 저하

      + 데이터 수정의 복잡성 : '개발팀'에 '최개발(ID:3)'을 추가하려면 어떻게 해야 할까?
        * 애플리케이션에서 '1,2' 를 읽어와서 문자열을 파싱하고, 3을 붙여서 '1,2,3' 으로 다시 업데이트해야 함
        * 반대로 회원을 팀에서 제외하는 로직은 더욱 복잡하며, 이런 로직은 오류가 발생하기 매우 쉬움
      + 참조 무결성 제약 불가 : member_ids 컬럼은 문자열(VARCHAR) 타입이므로 member 테이블의 member_id(BIGINT)를 참조하는 외래 키를 설정할 수 없음
        * 데이터베이스 레벨에서 데이터의 정합성을 보장할 방법이 사라지는 것
        * 예를 들어 존재하지 않는 회원 ID인 999 를 member_ids 에 '1,2,999' 와 같이 추가해도 데이터베이스는 막지 못함

  - 결론적으로, 컬럼 하나에 여러 값을 저장하는 방식은 관계형 데이터베이스의 장점을 모두 포기하는 최악의 설계이므로 절대로 사용해서는 안 됨

   - '일(1)' 쪽에 외래 키를 두면 발생하는 문제 3 : 여러 컬럼에 나누어 보관
     + 컬럼을 나누어 보관하는 경우 : member_id1, member_id2 ...
```sql
DROP TABLE IF EXISTS team_bad3;

CREATE TABLE team_bad3 (
 team_id BIGINT NOT NULL AUTO_INCREMENT,
 name VARCHAR(50) NOT NULL,
 member_id_1 BIGINT NULL, -- 첫 번째 팀원
 member_id_2 BIGINT NULL, -- 두 번째 팀원
 member_id_3 BIGINT NULL, -- 세 번째 팀원 ... 언제까지 추가할 것인가?

 PRIMARY KEY (team_id)
);
```
```sql
-- 데이터 삽입
INSERT INTO team_bad3(name, member_id_1, member_id_2)
VALUES ('개발팀', 1, 2); -- 김개발(ID:1), 박개발(ID:2);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f64d7f5b-d5aa-42d0-96a2-27ea879148c6">
</div>

   - 이 설계는 다음과 같은 심각한 문제들을 야기
      + 확장성의 부재 : 팀원이 3명에서 4명으로 늘어나면 어떻게 할 것인가?
        * ALTER TABLE 을 사용해 member_id_4 컬럼을 추가해야만 함
        * 팀원 수의 제한이 없는 비즈니스 요구사항을 전혀 반영하지 못함
        * 이는 팀원 수에 따라 테이블 구조가 계속 바뀌어야 하는 끔찍한 설계가 됨
      + 공간 낭비 : 대부분의 팀이 2 ~ 3명인데, 최대 팀원 수를 10명으로 가정하고 member_id_10까지 컬럼을 만들어두면, 나머지 NULL 컬럼들이 엄청난 공간을 낭비하게 됨
      + 데이터 검색의 어려움: "'박개발(ID:2)' 회원이 어느 팀 소속인가?"를 찾으려면 어떻게 해야 할까?
```sql
SELECT name
FROM team_bad3
WHERE member_id_1 = 2 OR member_id_2 = 2 OR member_id_3 = 2;
```
   - 이런 쿼리는 작성하기도 어렵고, 모든 member_id 컬럼을 확인해야 하므로 성능도 매우 나쁘며, 인덱스를 활용하기도 까다로움

4. '다(N)'쪽에 외래 키 두기
   - 반면, '다(N)' 쪽인 회원 테이블에 team_id 라는 컬럼 하나만 추가하면 모든 문제가 간단히 해결
   - team 테이블 (1 쪽) : 팀 정보는 각 팀마다 단 하나의 행만 차지하므로, 매우 깔끔하고 중복이 없음
<div align="center">
<img src="https://github.com/user-attachments/assets/0ed5cb31-db40-4a01-b711-c7ae080a4ba6">
</div>

   - member 테이블 (N 쪽) : 각 회원은 고유한 행을 가지며, team_id 컬럼에 자신이 속한 팀의 ID '단 하나'만 저장
<div align="center">
<img src="https://github.com/user-attachments/assets/3949c103-7b56-4054-83c8-abe53c18a519">
</div>

   - 각 회원은 자신이 속한 팀의 team_id만 저장하면 됨
     + 10명의 회원이든 100명의 회원이든 각자 자신의 행에 소속 팀 ID 하나만 가지면 되므로 데이터 구조가 매우 안정적이고 확장성 있게 유지

   - 이 구조의 핵심
     + 원자성 준수 : member 테이블의 team_id 컬럼에는 언제나 값 하나만 들어감
       * '김개발'의 소속팀은 1 이라는 값 하나로 명확하게 표현 : 이것이 관계형 데이터베이스의 가장 기본 원칙
     + 유연성: 팀에 소속되지 않은 '이기획' 같은 회원도 NULL 을 허용함으로써 표현할 수 있음
     + 확장성: '개발팀'에 신입사원 100명이 들어와도 team 테이블은 그대로이고, member 테이블에 100개의 행이 추가될 뿐, 테이블의 구조는 전혀 바꿀 필요가 없음
     + 이것이 바로 관계의 주인을 '다(N)' 쪽으로 정하고, 외래 키를 '다(N)' 쪽 테이블에 두는 이유

   - 💡 부모 테이블과 자식 테이블
     + 외래 키 관계를 이야기할 때, 우리는 두 테이블을 부모(Parent)와 자식(Child)으로 표현
     + 부모 테이블: 관계에서 '일(1)'에 해당하는 테이블로, 다른 테이블에 의해 참조되는 쪽으로, 기본 키(Primary Key)를 가지고 있음 (team 테이블)
     + 자식 테이블: 관계에서 '다(N)'에 해당하는 테이블로, 다른 테이블을 참조하는 쪽으로, 외래 키(Foreign Key)를 가지고 있음 (member 테이블)
     + 자식은 부모에게 의존 : 즉, member는 team에 의존하므로, 존재하지 않는 팀(team_id 가 없는)에 회원이 소속될수는 없음
  
   - 이처럼 데이터베이스는 외래 키 제약조건을 통해 부모 테이블에 존재하지 않는 데이터가 자식 테이블에 입력되는 것을 막아 참조 무결성(Referential Integrity)을 지켜줌

   - 다대일 관계 테이블을 설계
<div align="center">
<img src="https://github.com/user-attachments/assets/65ac24e8-2848-4306-9385-ee0934b33d0e">
</div>

   - 테이블 생성
     + team 테이블 생성 (관계에서 '1' 쪽, 부모 테이블) : 관계를 맺기 위해서는 참조의 대상이 되는 부모 테이블이 먼저 존재해야 함
```sql
DROP TABLE IF EXISTS member;
DROP TABLE IF EXISTS team;

CREATE TABLE team (
     team_id BIGINT NOT NULL AUTO_INCREMENT,
     name VARCHAR(50) NOT NULL,

     PRIMARY KEY (team_id),
     UNIQUE KEY uq_name (name)
);
```
   - member 테이블 생성 (관계에서 'N' 쪽, 자식 테이블) : 이제 team 을 참조하는 member 테이블 생성
```sql
DROP TABLE IF EXISTS member;

CREATE TABLE member (
     member_id BIGINT NOT NULL AUTO_INCREMENT,
     team_id BIGINT NULL, -- 팀에 소속되지 않은 회원이 있을 수 있다면 NULL 허용
     name VARCHAR(50) NOT NULL,

     PRIMARY KEY (member_id),
     CONSTRAINT fk_member_team FOREIGN KEY (team_id)
     REFERENCES team (team_id)
);
```
   - CONSTRAINT fk_member_team : 외래 키 제약조건에 fk_member_team 이라는 명시적인 이름을 부여하면, 오류 발생 시 원인을 파악하기 쉬움
   - FOREIGN KEY (team_id) REFERENCES team (team_id) : member 테이블의 team_id 가 team 테이블의 team_id 를 참조하도록 설정
     + 이로써 두 테이블 간의 관계가 완성
   - 데이터 삽입 및 결과 확인 : 데이터가 잘 입력되었는지 각 테이블을 조회해서 확인
```sql
-- 팀 데이터 삽입
INSERT INTO team(name) VALUES ('개발팀'), ('디자인팀');

-- 김개발, 박개발 -> 개발팀 (team_id=1)
INSERT INTO member(team_id, name)
VALUES (1, '김개발'), (1, '박개발');

-- 최디자인 -> 디자인팀 (team_id=2)
INSERT INTO member(team_id, name)
VALUES (2, '최디자인');

-- 이기획 -> 팀 없음 (team_id=NULL)
INSERT INTO member(team_id, name)
VALUES (NULL, '이기획');
```
   - 각 테이블의 데이터를 조회
   - team 테이블 조회
```sql
SELECT *
FROM team;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/97012b72-3a95-4cd1-b274-be93801bdf4f">
</div>

   - member 테이블 조회
```sql
SELECT *
FROM member;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2b94cd8d-9366-4c74-b9d2-cb1e4025d419">
</div>

   - 결과를 보면 member 테이블의 team_id 컬럼이 외래 키 역할을 하면서 각 회원이 어떤 팀에 속해있는지 명확하게 보여줌
   - '김개발'과 '박개발'은 team_id 가 1인 '개발팀'에, '최디자인'은 team_id 가 2인 '디자인팀'에 소속된 것을 확인할 수 있음
   - 이처럼 '다(N)' 쪽인 member 테이블에 외래 키를 두는 것이 일대다 관계를 표현하는 올바른 방법
