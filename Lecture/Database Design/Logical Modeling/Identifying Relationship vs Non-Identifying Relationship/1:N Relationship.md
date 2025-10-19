-----
### 식별 관계 vs 비식별 관계 - 일대다(1:N)
-----
1. 일대다 관계에서 발생하는 식별 관계와 비식별 관계
   - 게시글(BOARD)과 댓글(COMMENT)의 관계의 예시 : 나의 게시글에는 여러 댓글이 달릴 수 있음 (1:N)

2. 비식별 관계 (Non-identifying) : 이 방식은 자식 테이블인 COMMENT가 자신만의 고유한 기본 키(comment_id)를 가지는 구조
<div align="center">
<img src="https://github.com/user-attachments/assets/cd9ddfe9-def4-4cb0-834b-bf5c5b18dd49">
</div>

   - ERD에서 비식별 관계는 점선으로 표시하며, 식별 관계는 실선으로 표시
   - 테이블 설계
```sql
DROP TABLE IF EXISTS comment_non_identifying;
DROP TABLE IF EXISTS board_non_identifying;

CREATE TABLE board_non_identifying (
   board_id BIGINT NOT NULL AUTO_INCREMENT,
   title VARCHAR(255) NOT NULL,
   PRIMARY KEY (board_id)
);
CREATE TABLE comment_non_identifying (
   comment_id BIGINT NOT NULL AUTO_INCREMENT, -- 독립적인 PK
   board_id BIGINT NOT NULL, -- 일반 컬럼 + FK
   content TEXT NOT NULL,

   PRIMARY KEY (comment_id),
   CONSTRAINT fk_comment_board_non FOREIGN KEY (board_id)
   REFERENCES board_non_identifying (board_id)
);
```
   - comment_non_identifying 테이블은 comment_id라는 독립적인 대리 키(Surrogate Key)를 PK로 사용
   - board_id (FK)는 어떤 게시글에 달린 댓글인지를 나타내는 단순 외래 키일 뿐, PK의 일부가 아님
   - 장점
      + 단순하고 명확함 : comment_id 하나만 알면 특정 댓글을 조회, 수정, 삭제할 수 있으며, PK가 단순하여 관리가 매우 쉬움
      + 유연함 : 만약 비즈니스 로직이 변경되어 댓글을 다른 게시글로 옮겨야 하는 상황이 생기면, board_id (FK) 값만 UPDATE 하면 됨 (관계가 느슨하게 연결되어 있어 변화에 유연)
      + ORM 친화적 : JPA / Hibernate 같은 ORM 기술은 이런 단순한 대리 키 구조에서 가장 자연스럽고 효율적으로 동작

   - 단점 : PK만 봐서는 부모와의 관계를 직관적으로 알기 어려움 (큰 단점은 아님)
   - 데이터 입력 및 조회
```sql
-- 데이터 입력
INSERT INTO board_non_identifying (title)
VALUES ('첫 번째 게시글'); -- board_id:1
INSERT INTO board_non_identifying (title)
VALUES ('두 번째 게시글'); -- board_id:2

INSERT INTO comment_non_identifying (board_id, content)
VALUES (1, '1번 글의 첫 댓글입니다.'); -- comment_id: 1
INSERT INTO comment_non_identifying (board_id, content)
VALUES (1, '1번 글의 두 번째 댓글입니다.'); -- comment_id: 2
INSERT INTO comment_non_identifying (board_id, content)
VALUES (2, '2번 글의 첫 댓글입니다.'); -- comment_id: 3
```
<div align="center">
<img src="https://github.com/user-attachments/assets/0ef6058f-ead3-4448-86d5-f06993a7f2dc">
</div>

   - 결과를 보면 알 수 있듯이, 각 댓글은 board_id 와 관계없이 comment_id 라는 고유한 값(1, 2, 3)으로 완벽하게 식별
   - 댓글을 조회하거나 수정할 때 comment_id=2와 같이 하나의 값만 사용하면 되므로 매우 직관적

3. 식별 관계 (Identifying)
   - 자식 테이블인 COMMENT가 부모의 기본 키(board_id)를 물려받아 자신의 기본 키의 일부로 삼음
<div align="center">
<img src="https://github.com/user-attachments/assets/a2f8e5b2-63a7-44cf-8785-19c264d3d975">
</div>

   - ERD에서 비식별 관계는 점선으로 표시하며, 식별 관계는 실선으로 표시
   - 테이블 설계
```sql
DROP TABLE IF EXISTS comment_identifying;
DROP TABLE IF EXISTS board_identifying;

CREATE TABLE board_identifying (
     board_id BIGINT NOT NULL AUTO_INCREMENT,
     title VARCHAR(255) NOT NULL,
     PRIMARY KEY (board_id)
);

CREATE TABLE comment_identifying (
     board_id BIGINT NOT NULL, -- PK의 일부 + FK
     comment_no BIGINT NOT NULL, -- PK의 일부, 해당 board_id 내에서 순차 증가
     content TEXT NOT NULL,

     PRIMARY KEY (board_id, comment_no), -- 복합 기본 키 (Composite PK)
     CONSTRAINT fk_comment_board_iden FOREIGN KEY (board_id)
     REFERENCES board_identifying (board_id)
);
```

   - comment_identifying 테이블의 PK는 (board_id, comment_no) 두 개의 컬럼으로 구성된 복합키
   - 일대다 관계이므로 부모로부터 받은 PK인 board_id만으로는 PK를 구성할 수 없음 : comment_no 같은 추가 정보가 필요
   - board_id는 PK이면서 동시에 board_identifying 테이블을 참조하는 FK
   - 참고
     + comment_no는 댓글 순서에 맞추어 애플리케이션에서 번호를 만들어야 함(매번)
     + 데이터베이스가 제공하는 자동 증가값(auto increment)을 사용할 수 없음

   - 장점
     + PK 자체가 비즈니스 의미를 가짐 : (board_id=10, comment_no=3) 이라는 PK는 '10번 게시글의 3번째 댓글'이라는 명확한 의미를 가짐
     + 데이터 정합성 강화 : 이 구조에서는 board_id 없이 comment가 존재할 수 없다는 것이 명확하게 표현
     + 쿼리 최적화 가능성 : 게시글과 댓글을 항상 같이 조회하는 경우, 복합키 인덱스가 효율적으로 사용될 수 있음
   - 단점
     + 복잡함 : 복합키는 관리하기가 번거로움 (조회, 수정, 삭제 시 항상 두 개의 키 값을 모두 사용해야 함)
     + 유연성이 없음 : 부모의 PK가 자식 PK의 일부가 되므로, 댓글을 다른 게시글로 옮기는 등의 비즈니스 요구사항 변경에 대응하기 매우 어려움 (PK 값 자체를 변경해야 하기 때문임)
     + 확장성이 떨어짐 : 만약 댓글에 reply 라는 대댓글 기능이 추가된다면, PK 구조가 계속해서 길어지고 복잡해지는 문제가 발생
     + ORM 매핑의 복잡성 : ORM에서 복합키를 매핑하려면 별도의 식별자 클래스를 만드는 등 추가적인 작업이 필요
   - 데이터 입력 및 조회
```sql
-- 데이터 입력
INSERT INTO board_identifying (title)
 VALUES ('첫 번째 게시글'); -- board_id: 1
INSERT INTO board_identifying (title)
VALUES ('두 번째 게시글'); -- board_id: 2

-- 애플리케이션 레벨에서 comment_no를 계산해서 넣어주어야 함
INSERT INTO comment_identifying (board_id, comment_no, content)
VALUES (1, 1, '1번 글의 첫 댓글입니다.');
INSERT INTO comment_identifying (board_id, comment_no, content)
VALUES (1, 2, '1번 글의 두 번째 댓글입니다.');
INSERT INTO comment_identifying (board_id, comment_no, content)
VALUES (2, 1, '2번 글의 첫 댓글입니다.');
```
<div align="center">
<img src="https://github.com/user-attachments/assets/6e2ceb1d-a15a-47d7-a841-3711b520d875">
</div>

   - 여기서는 comment_no 컬럼만 보면 1이라는 값이 중복
   - 하지만 기본 키는 (board_id, comment_no) 두개를 합친 복합키이므로 (1, 1), (1, 2), (2, 1) 은 모두 고유한 값으로 인정
   - 특정 댓글을 조회하려면 WHERE board_id = 1 AND comment_no = 2와 같이 항상 두 개의 키 값을 조건으로 사용해야 함
