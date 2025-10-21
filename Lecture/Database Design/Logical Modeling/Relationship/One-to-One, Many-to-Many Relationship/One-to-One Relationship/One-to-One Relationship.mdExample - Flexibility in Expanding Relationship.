-----
### 일대일(1:1) 관계 - 관계 확장의 유연성
-----
1. 관계 확장의 유연성 (1:1 → 1:N)
   - 비즈니스 요구사항은 계속 변함 : "처음에는 하나만 가능했는데, 이제 여러 개가 필요해요"라는 요구는 매우 흔함
   - 이 설계는 이러한 변화에 매우 유연하게 대처할 수 있음
   - 예를 들어, 처음에는 게시글(board) 하나에 첨부파일(upload_file)을 하나만 등록할 수 있도록 설계했다고 가정
     + 이후 유니크 제약조건을 제거하면 1:1 → 1:N으로 구조를 변경할 수 있음
     + 처음에는 하나의 게시판에 반드시 하나의 파일만 업로드 할 수 있다고 가정
<div align="center">
<img src="https://github.com/user-attachments/assets/1d418d82-01b6-4b24-ac54-614436bda58a">
</div>

   - upload_file.board_id (FK) : 유니크 제약조건

2. 초기 1:1 관계 설계 (upload_file 테이블)
```sql
DROP TABLE IF EXISTS upload_file;
DROP TABLE IF EXISTS board;

-- 게시글 테이블
CREATE TABLE board (
   board_id BIGINT NOT NULL AUTO_INCREMENT,
   title VARCHAR(255) NOT NULL,
   content TEXT NULL,
   PRIMARY KEY (board_id)
);

CREATE TABLE upload_file (
   upload_file_id BIGINT NOT NULL AUTO_INCREMENT,
   board_id BIGINT NOT NULL,
   upload_file_name VARCHAR(255) NOT NULL,

   PRIMARY KEY (upload_file_id),
   UNIQUE KEY uq_board_id (board_id), -- 이 제약조건으로 1:1 관계를 보장
   CONSTRAINT fk_upload_file_board FOREIGN KEY (board_id) REFERENCES board(board_id)
);
```
   - 실무에서는 파일의 위치 또는 실제 파일의 내용이 필요하자만, 여기서는 생략
   - upload_file 테이블을 확인
```sql
desc upload_file;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f96d1cc4-5ba8-41db-8fbc-7a5b0fb7e20b">
</div>

  - board_id가 중복을 허용하지 않는 유니크 키(UNI , Unique Key, 유니크 인덱스)로 만들어진 것을 확인

3. 1:1 관계 데이터 추가 및 검증
   - 실제 데이터를 추가하며 UNIQUE 제약조건이 어떻게 동작하는지 확인 : 먼저 게시글을 하나 작성
```sql
INSERT INTO board (title, content)
VALUES ('1:1 관계 테스트', '하나의 파일만 첨부됩니다.');
```
   - board_id 가 1인 게시글이 생성되었다고 가정 : 이제 이 게시글에 첨부파일을 추가
```sql
INSERT INTO upload_file (board_id, upload_file_name)
VALUES (1,'document.pdf');
```
  - 여기까지는 문제없이 실행 : 하지만 동일한 board_id 로 파일을 하나 더 추가하려고 하면 어떻게 될까?
```sql
INSERT INTO upload_file (board_id, upload_file_name)
VALUES (1, 'image.jpg');
```
  - 실행 결과
```
Error Code: 1062. Duplicate entry '1' for key 'upload_file.uq_board_id'
```
   - 데이터베이스가 uq_board_id 라는 유니크 제약조건을 위반했기 때문에 INSERT 를 거부하는 것을 볼 수 있음
   - 이로써 board와 upload_file은 완벽한 1:1 관계를 유지하게 됨
   - 현재까지의 관계를 JOIN 으로 조회
```sql
SELECT
     b.board_id,
     b.title,
     f.upload_file_id,
     f.upload_file_name
FROM board b
JOIN upload_file f ON b.board_id = f.board_id
ORDER BY f.upload_file_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/e037eedd-4281-479d-bd8d-11aa5fdecd4c">
</div>
  
   - 참고 : 여기서 board_id , upload_file_id 의 값은 1로 같지만, 서로 다른 ID 값

 4. 그런데 서비스가 성장하여 게시글 하나에 파일을 여러 개 첨부하고 싶다는 요구사항이 발생 : 이 설계에서는 유니크 제약조건만 제거하면 됨
   - 단 몇 줄의 DDL 변경으로 이 요구사항을 완벽하게 만족시킬 수 있음
```sql
-- 1단계: 외래 키(Foreign Key) 제약조건 삭제 (MySQL에서는 인덱스 변경 전 FK를 먼저 삭제해야 함)
ALTER TABLE upload_file DROP FOREIGN KEY fk_upload_file_board;

-- 2단계: 기존 UNIQUE 인덱스 삭제
ALTER TABLE upload_file DROP INDEX uq_board_id;

-- 3단계: 외래 키 제약조건 다시 추가 (유니크 인덱스 -> 일반 인덱스)
ALTER TABLE upload_file
ADD CONSTRAINT fk_upload_file_board FOREIGN KEY (board_id)
REFERENCES board (board_id);
```
   - upload_file 테이블을 확인
```sql
desc upload_file;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a8db06b6-cb71-42ba-954a-35535454b430">
</div>

  - board_id의 Key가 UNI에서 MUL(Multiple Key, 중복을 허용하는 일반 인덱스)로 변경된 것을 확인
  - 참고로 MySQL은 인덱스가 없는 경우 외래 키 제약조건을 추가하면 일반 인덱스가 자동으로 만들어짐
  - 이제 upload_file 테이블의 board_id에는 중복된 값이 들어갈 수 있으므로, 자연스럽게 board와 upload_file은 일대다(1:N) 관계 : 테이블 구조의 근본적인 변경 없이 관계의 카디널리티를 유연하게 확장할 수 있는 것
<div align="center">
<img src="https://github.com/user-attachments/assets/a87016fd-69d3-4a53-8242-1c63d0345e3f">
</div>

   - upload_file.board_id (FK) : 유니크 제약조건 없음

5. 1:N 관계로 전환 후 데이터 추가 및 검증
   - UNIQUE 제약조건이 사라졌으니, 앞서 실패했던 두 번째 파일 추가를 다시 시도
```sql
INSERT INTO upload_file (board_id, upload_file_name)
VALUES (1, 'image.jpg');

INSERT INTO upload_file (board_id, upload_file_name)
VALUES (1, 'data.csv');
```
   - 이제는 아무런 오류 없이 데이터가 잘 저장
   - JOIN을 통해 관계를 다시 확인
```sql
SELECT
     b.board_id,
     b.title,
     f.upload_file_id,
     f.upload_file_name
FROM board b
JOIN upload_file f ON b.board_id = f.board_id
ORDER BY f.upload_file_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/47593b30-9849-4d4b-ac90-3c177d626c0b">
</div>

   - 하나의 게시글(board_id = 1)에 여러 파일이 성공적으로 연결된 것을 볼 수 있음
   - 이처럼 보조 테이블에 외래 키를 두면 변화에 유연하게 대처할 수 있음
