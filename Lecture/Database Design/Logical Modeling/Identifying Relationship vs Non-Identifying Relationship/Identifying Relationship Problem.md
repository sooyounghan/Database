-----
### 식별 관계의 문제점
-----
1. 식별 관계는 부모의 식별자를 자식이 물려받는다는 점에서 논리적으로 명확해 보일 수 있음
   - '게시글 없는 댓글은 없다'와 같은 비즈니스 규칙을 데이터 구조에 직접적으로 표현하는 것처럼 느껴지기 때문임
   - 하지만 이런 강한 결합(Tight Coupling)은 실무적인 관점에서 여러 심각한 문제점을 만드는데, 이는 애플리케이션의 유연성과 확장성을 크게 저해하는 원인

2. 식별 관계의 핵심적인 문제점
   - 유연성 부족 : 부모와 자식이 PK로 강하게 묶여 있기 때문에, 한번 맺어진 관계를 변경하는 것이 매우 어려움
     + 예를 들어 댓글을 다른 게시글로 옮기는 등의 비즈니스 요구사항 변경에 유연하게 대처할 수 없음
   - 기본 키(PK) 컬럼의 전파와 복잡성 증가 : 관계의 깊이가 깊어질수록(자식, 손자 테이블로 내려갈수록) 부모의 PK가 계속해서 누적되어 자식 테이블의 PK가 비대해지고 복잡해짐
     + 이는 SQL 쿼리와 JOIN을 어렵게 만들고, ORM(Object-Relational Mapping) 기술 사용의 복잡성을 가중
   - 이러한 단점들 때문에 현대적인 데이터베이스 설계에서는 대부분 비식별 관계를 사용

3. 유연성 부족 : 관계 변경의 어려움
   - 이 단점은 식별 관계를 실무에서 기피하는 가장 결정적인 이유 중 하나
   - "1번 게시글의 2번 댓글(board_id=1, comment_no=2)을 관리자가 2번 게시글로 옮겨야 하는 상황"을 가정
     + 비식별 관계라면, 이 작업은 매우 간단함 : 댓글의 고유 식별자인 comment_id는 그대로 유지한 채, 소속 게시글을 가리키는 외래 키 board_id의 값만 변경
```sql
-- comment_id=2인 댓글을 board_id=2로 옮김
UPDATE comment_non_identifying
SET board_id = 2
WHERE comment_id = 2;
```
   - 댓글의 '정체성(comment_id=2)'은 그대로 유지되면서 '소속(board_id)'만 깔끔하게 변경 : 이것이 바로 '느슨한 결합(Loosely Coupled)'이 주는 유연성

     + 식별 관계라면, 하지만 식별 관계에서는 문제가 완전히 달라짐
       + 댓글의 기본 키는 (board_id, comment_no)라는 복합키
       + board_id를 1에서 2로 바꾸려는 시도는 기본 키(PK) 자체를 수정하는 것
       + PK는 데이터의 무결성을 지키는 가장 중요한 약속이며, 한 번 정해지면 변하지 않는다는 불변성(Immutability)을 가정하는 것이 현대 데이터베이스 설계의 기본 원칙
       + 만약 다른 테이블에서 (board_id=1, comment_no=2) 라는 복합키를 외래 키로 참조하고 있었다면, 이 PK를 변경하는 순간 모든 참조가 끊어지는 '데이터 무결성 붕괴'가 발생
       + 이처럼 식별 관계는 부모와 자식을 매우 강하게 결합(Tightly Coupled)시킴

   - 한번 맺어진 관계는 변경하기가 극히 어려움
   - 반면, 비식별 관계는 독립적인 식별자를 통해 느슨한 결합(Loosely Coupled)을 유지하므로, 미래에 발생할 수 있는 비즈니스 로직 변경에 훨씬 유연하게 대처할 수 있음

4. 식별관계에서 자식, 손자 테이블이 추가되는 경우 문제점 예제
   - 식별 관계의 또 다른 큰 문제점은 관계의 깊이가 깊어질수록, 즉 부모-자식-손자-증손자로 이어지는 데이터 모델이 만들어질 때 그 문제점이 명확하게 드러남 : 이를 'PK 컬럼의 전파' 또는 'PK 오염' 이라고도 부름
   - 앞서 다룬 게시글과 댓글의 관계를 확장하여, 댓글에 '대댓글'이 달리는 계층 구조 예시
      + board_identifying (게시글) : 최상위 부모
      + comment_identifying (댓글) : 게시글에 속하는 자식
      + reply_identifying (대댓글) : 댓글에 종속되는 손자
   - 1단계 : board_identifying 테이블 생성 (부모)
      + 부모 테이블은 board_id를 PK로 가짐
```sql
DROP TABLE IF EXISTS reply_identifying;
DROP TABLE IF EXISTS comment_identifying;
DROP TABLE IF EXISTS board_identifying;

CREATE TABLE board_identifying (
   board_id BIGINT NOT NULL AUTO_INCREMENT,
   title VARCHAR(255) NOT NULL,

   PRIMARY KEY (board_id)
);
```
   - PK : (board_id)
   - 2단계 : comment_identifying 테이블 생성 (자식)
<div align="center">
<img src="https://github.com/user-attachments/assets/5a93213d-4893-47cd-941e-144df199f220">
</div>

   - 자식인 comment_identifying은 부모인 board_identifying의 PK를 물려받아 자신의 PK 일부로 삼음
```sql
CREATE TABLE comment_identifying (
     board_id BIGINT NOT NULL, -- 부모 PK를 상속 (PK + FK)
     comment_no BIGINT NOT NULL, -- 해당 board_id 내에서의 고유 번호
     content TEXT NOT NULL,

     PRIMARY KEY (board_id, comment_no), -- 복합키
     CONSTRAINT fk_comment_to_board FOREIGN KEY (board_id)
     REFERENCES board_identifying(board_id)
);
```
   - PK : (board_id, comment_no)
   - 3단계: reply_identifying 테이블 생성 (손자)
<div align="center">
<img src="https://github.com/user-attachments/assets/652016fd-f77f-46db-80b4-4e3285c917e1">
</div>

   - 이제 손자인 reply_identifying을 생성 : 이 테이블은 자신의 부모인 comment_identifying의 PK를 상속받아야 함
   - 그런데 comment_identifying 의 PK는 (board_id, comment_no) 복합키이므로, 따라서 이 두 개의 컬럼을 모두 물려받아야 함
```sql
CREATE TABLE reply_identifying (
     board_id BIGINT NOT NULL, -- 조상 PK를 상속 (PK + FK)
     comment_no BIGINT NOT NULL, -- 부모 PK를 상속 (PK + FK)
     reply_no BIGINT NOT NULL, -- 해당 comment_no 내에서의 고유 번호
     content TEXT NOT NULL,

     PRIMARY KEY (board_id, comment_no, reply_no), -- 더 길어진 복합키
     CONSTRAINT fk_reply_to_comment FOREIGN KEY (board_id, comment_no)
     REFERENCES comment_identifying (board_id, comment_no)
);
```
   - PK : (board_id, comment_no, reply_no)

   - PK의 비대화 : 단지 3단계의 계층 구조일 뿐인데, 손자 테이블의 PK는 3개의 컬럼으로 구성된 복합키가 되어버렸음
     + 만약 계층이 더 깊어진다면 PK는 계속해서 길어질 것
   - 복잡한 SQL : 특정 대댓글을 조회하려면 3개의 키 값을 모두 알아야 함
```sql
SELECT *
FROM reply_identifying
WHERE board_id = 1 AND comment_no = 10 AND reply_no = 3;
```
   - 어려운 JOIN : reply_identifying 테이블과 다른 테이블을 JOIN 할 때, JOIN 조건에 3개의 컬럼을 모두 명시해야 하는 경우가 발생하여 쿼리가 길고 복잡해짐
   - ORM 복잡 : JPA 같은 ORM에서는 이 복합키를 다루기 위해 매우 번거로운 추가 작업(예) @EmbeddedId, @IdClass)이 필요

   - 비식별 관계와 손자 테이블
     + 반면, 이 구조를 비식별 관계로 설계했다면 모든 테이블은 board_id, comment_id, reply_id 라는 단순한 단일 PK를 가졌을 것
     + 손자 테이블인 reply는 부모의 PK인 comment_id 하나만 FK로 가지면 됨 : 훨씬 더 단순하고 유연하며 관리하기 쉬운 구조
<div align="center">
<img src="https://github.com/user-attachments/assets/3e4640f0-3294-4d4c-8b8c-ff6332e91a89">
</div>

```sql
DROP TABLE IF EXISTS reply_non_identifying;

CREATE TABLE reply_non_identifying (
   reply_id BIGINT NOT NULL AUTO_INCREMENT, -- 독립적인 PK
   comment_id BIGINT NOT NULL, -- 부모를 참조하는 FK
   content TEXT NOT NULL,

   PRIMARY KEY (reply_id),
   CONSTRAINT fk_reply_comment_non FOREIGN KEY (comment_id)
   REFERENCES comment_non_identifying(comment_id)
);
```

5. 참고 - 실무이야기
   - 과거에는 회원의 식별을 위해 주민등록번호를 수집하고, 그 유일성을 믿어 PK로 사용하곤 했는데, 이것이 식별 관계와 만나면 끔찍한 결과를 낳음
   - 가령, member (회원) 테이블이 주민등록번호(ssn)를 PK로 사용했다고 가정
     + 그리고 이 회원의 활동 내역을 기록하는 activity_log (자식)와 log_detail (손자) 테이블이 식별 관계로 엮여 있음
       * member PK : (ssn)
       * activity_log PK : (ssn, log_dt)
       * log_detail PK : (ssn, log_dt, detail_sequence)
     + ssn 컬럼이 손자 테이블까지 계속 전파되어 PK의 일부가 된 것을 볼 수 있음
     + 초기에는 아무 문제가 없어 보였지만, 정부에서 개인정보보호법을 강화하여 주민등록번호를 원칙적으로 수집 및 저장을 금지하는 정책을 발표 : 이제 데이터베이스에서 주민등록번호를 완전히 삭제해야 함
     + 식별 관계였기 때문에 벌어진 끔찍한 변경 작업 ssn은 단순한 컬럼이 아니라 세 테이블 모두의 '신원(Identity)' 그 자체
     + ALTER TABLE member DROP COLUMN ssn; 같은 간단한 명령은 불가능 : PK를 삭제할 수는 없기 때문이며, 이 문제를 해결하려면 다음과 같이 해야함
       * member 테이블에 member_id라는 새로운 대리 키 컬럼을 추가
       * activity_log, log_detail 테이블에도 member_id 컬럼을 추가
       * 기존 데이터에 대해 ssn을 기준으로 모든 테이블의 member_id를 채워 넣는 마이그레이션 스크립트를 실행
       * 세 테이블의 기본 키(PK)와 외래 키(FK) 제약조건을 모두 삭제하고, member_id 기반의 새로운 PK와 FK를 다시 정의
       * 애플리케이션 코드에서 ssn을 기준으로 작성된 모든 SQL 쿼리와 로직을 member_id 기반으로 수정
     + 이 과정은 서비스의 장시간 중단을 유발할 수 있으며, 데이터가 유실되거나 꼬일 위험이 매우 큰, 그야말로 '끔찍한' 작업

   - 만약 처음부터 비식별 관계였다면?
     + member 테이블 : PK는 member_id, ssn은 별도의 UNIQUE 컬럼
     + activity_log 테이블 : PK는 log_id, FK로 member_id를 가짐
     + log_detail 테이블 : PK는 detail_id, FK로 log_id 를 가짐
     + 이 구조였다면 변경에 대응하는 작업은 믿을 수 없을 만큼 간단해짐
       * member 테이블에서 ssn 컬럼을 사용하는 로직(예) 본인 인증)을 다른 방식으로 바꿈
       * ALTER TABLE member DROP COLUMN ssn; 명령을 실행
     + 자식, 손자 테이블은 member_id로 관계를 맺고 있으므로 아무런 영향을 받지 않음
     + 핵심 식별자가 비즈니스 규칙(주민등록번호)과 분리되어 있었기 때문에, 규칙의 변경이 구조 전체를 흔들지 않은 것
     + 이는 비식별 관계의 유연성이 얼마나 중요한지를 보여주는 실무의 대표적인 사례

6. 정리 : 왜 비식별 관계를 선택해야 하는가?
    - 식별 관계는 '관계'와 '비즈니스 규칙'(주민등록번호, 게시글 번호 등)을 테이블의 '신원(PK)' 에 직접 묶어버림
      + 이는 논리적으로는 명확해 보이지만, 비즈니스 규칙이나 데이터의 소속 관계가 변경될 때 PK 자체를 흔들어 시스템 전체에 치명적인 영향을 줌
    - 반면, 비식별 관계는 이들을 명확히 분리
      + 신원(PK) : 비즈니스와 무관한 대리 키( id )가 담당 - 이 값은 한번 정해지면 절대 변하지 않음
      + 관계(FK) : 외래 키(parent_id)가 담당 - 관계가 바뀌면 이 값만 수정하면 됨
      + 비즈니스 데이터 : 일반 컬럼이 담당 - 규칙이 바뀌면 해당 컬럼만 수정하거나 삭제하면 됨

    - 이처럼 각자의 역할과 책임을 명확히 분리했기 때문에, 비즈니스 규칙이 바뀌거나(주민번호 저장 금지) 관계가 바뀌어도(댓글 이동), 테이블의 본질적인 신원(id)은 흔들리지 않음
    - 따라서 변경의 영향 범위가 최소화되고 시스템을 유연하고 견고하게 만듬
    - 이런 이유로 현대 애플리케이션 개발에서는 대부분의 경우 유연성과 확장성을 위해 비식별 관계를 선택하는 것이 일반적인 모범 사례(Best Practice)

7. 💡 대리 키 + 비식별 관계
   - 현대 애플리케이션 개발에서는 '기본적으로 모든 관계는 비식별 관계로 설계하고, PK는 비즈니스와 무관한 대리 키를 사용한다' 는 원칙을 따르는 것이 현명한 선택
   - 이것이 변화에 유연하게 대처하고 유지보수하기 좋은 시스템을 만드는 현대적인 설계 방법

8. 대리 키-PK, 자연 키-UNIQUE, 관계-비식별 : 현대적인 데이터베이스 설계는 대리 키-PK, 자연 키-UNIQUE, 관계-비식별 방식이 사실상 표준
