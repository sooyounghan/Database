-----
### 데이터 타입 - 날짜와 시간 타입
-----
1. 날짜 및 시간 타입
   - 날짜와 시간 정보를 저장하는 데 사용되는 타입은 매우 중요 : 문자열로 저장하면 날짜 계산이나 비교가 매우 어렵기 때문임
   - DATE : 날짜만 저장, 'YYYY-MM-DD' 형식 (예) '2025-08-21')
   - DATETIME : 날짜와 시간을 함께 저장, 'YYYY-MM-DD HH:MI:SS' 형식 (예) '2025-08-21 11:30:00')
   - TIME : 시간만 저장, 'HH:MI:SS' 형식

2. 실무 가이드
   - 회원 가입일, 주문일, 게시물 작성일 등 대부분의 경우 DATETIME을 사용
   - 생년월일처럼 시간 정보가 필요 없는 경우에는 DATE를 사용
   - 특별한 이유가 없다면 TIMESTAMP 대신에 DATETIME을 사용하는 것이 더 직관적이고 안전
     + TIMESTAMP는 시간대 변환 / 범위(1970~2038) 제약이 있어 비즈니스 시각 기록은 DATETIME이 일반적으로 안전

   - 생성일과 수정일은 기본 : 실무에서는 대부분의 테이블에 created_at (생성일시)과 updated_at (수정일시) 컬럼을 추가하는데, 이 데이터는 나중에 문제 추적이나 데이터 분석에 매우 유용하게 사용

3. 생성일과 수정일 자동화
   - created_at과 updated_at은 매우 중요하지만, 데이터를 삽입하거나 수정할 때마다 개발자가 직접 NOW() 함수를 사용해서 시간을 넣어주는 것은 번거롭고 실수를 유발하기 쉬움
   - 만약 깜빡하고 값을 넣지 않으면 데이터가 누락될 것
   - 데이터베이스는 이런 반복적인 작업을 자동화할 수 있는 매우 강력하고 편리한 기능을 제공
      + DEFAULT CURRENT_TIMESTAMP : 컬럼에 기본값을 현재 시간으로 설정 (데이터가 처음 삽입될 때, 해당 컬럼에 값을 명시하지 않으면 데이터베이스가 자동으로 현재 시간을 기록)
      + ON UPDATE CURRENT_TIMESTAMP : 해당 로우(row)의 데이터가 수정될 때마다, 자동으로 현재 시간으로 값을 갱신

   - 이 두 가지 옵션을 사용하면 created_at과 updated_at을 개발자가 전혀 신경 쓰지 않아도 데이터베이스가 알아서 관리해줌

4. 예제 : 생성일과 수정일이 자동 관리되는 게시판 테이블
   - 게시판 테이블을 예시로 DDL을 작성
```sql
DROP TABLE IF EXISTS board_sample;

CREATE TABLE board_sample (
     board_id BIGINT NOT NULL AUTO_INCREMENT,
     title VARCHAR(255) NOT NULL,
     content TEXT NULL,

     created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
     updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

     PRIMARY KEY (board_id)
);
```
   - created_at : DEFAULT CURRENT_TIMESTAMP를 적용해서, 데이터가 생성될 때의 시간이 자동으로 기록되도록 함
   - updated_at : DEFAULT CURRENT_TIMESTAMP와 ON UPDATE CURRENT_TIMESTAMP를 모두 적용
     + 데이터가 처음 생성될 때는 created_at 과 동일한 시간이 기록되고, 이후에 해당 데이터가 수정될 때마다 수정된 시점의 시간으로 자동으로 갱신

   - 자동화 기능 확인하기 : 실제로 데이터베이스가 어떻게 자동으로 시간을 기록하는지 확인
      + 데이터 삽입 : created_at과 updated_at 컬럼을 제외하고 INSERT 문을 실행
```sql
INSERT INTO board_sample (title, content)
VALUES ('첫 번째 게시글', '게시글 내용입니다.');
```
   - 삽입 결과 확인 : 데이터를 조회해서 created_at과 updated_at에 값이 잘 들어갔는지 확인
```sql
SELECT board_id, title, created_at, updated_at
FROM board_sample;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/b53e8375-a031-41e4-95b8-cdaf0ab9c939">
</div>

   - 결과를 보면, INSERT 문에 시간을 전혀 명시하지 않았는데도 created_at과 updated_at에 정확한 생성 시각이 기록된 것을 확인할 수 있음

   - 데이터 수정 : 방금 삽입한 게시글의 내용을 수정
```sql
UPDATE board_sample
SET content = '게시글 내용이 수정되었습니다.'
WHERE board_id = 1;
```
   - 수정 결과 확인 : 다시 데이터를 조회해서 updated_at이 변경되었는지 확인
```sql
SELECT board_id, title, created_at, updated_at
FROM board_sample;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/da0b8083-9103-4a6e-b9aa-377fc12b3a5c">
</div>

   - created_at은 처음 생성된 시간 그대로 유지되고, updated_at 컬럼만 데이터가 수정된 시간으로 깔끔하게 변경
   - 이렇게 데이터베이스의 자동화 기능을 활용하면 데이터가 언제 변경되었는지 쉽게 추적할 수 있음 : 개발자는 비즈니스 로직에 더 집중할 수 있게 됨
   - 실무에서는 거의 모든 테이블에 이 방식을 적용

5. 💡 비즈니스 날짜 vs 자동 생성 날짜
   - 테이블의 member 테이블에는 회원이 가입한 시점을 기록하는 created_at이 존재
   - orders 테이블에는 ordered_at과 created_at이 둘 다 존재
   - 이 둘의 차이는 실무에서 매우 중요
      + ordered_at : 회원이 '주문'이라는 비즈니스 행위를 한 시점
      + created_at : 이 주문 데이터가 우리 데이터베이스 테이블에 '생성'된 시점
   - 대부분의 경우 두 시간은 거의 동일하겠지만, 데이터 마이그레이션이나 지연 처리 등으로 인해 달라질 수 있음
   - 이렇게 목적에 따라 컬럼 이름을 명확히 구분하면 데이터의 의미를 혼동 없이 정확하게 파악할 수 있음

6. 실무 팁
   - created_at은 데이터가 처음 데이터베이스에 저장된 물리적인 시간을 기록하는 용도
   - ordered_at와 같은 컬럼은 주문 접수, 결제 완료 등 비즈니스 이벤트가 발생한 논리적인 시간을 기록하는 용도로 구분
   - 구분함으로써, 시스템을 더 정교하게 관리할 수 있다

7. 용어 사전 추가 - created at, updated at : 이런 규칙을 확정했다면 용어 사전에 추가해서 관리하는 것이 좋음
<div align="center">
<img src="https://github.com/user-attachments/assets/7bc166cc-bf36-4e71-879c-81346421be66">
</div>

8. 기타 타입
   - BOOLEAN (또는 BOOL) : 내부적으로는 TINYINT(1)로 처리
     + TRUE는 1, FALSE는 0으로 저장
     + 상품의 전시 여부, 회원의 탈퇴 여부 등 참/거짓 상태를 나타낼 때 유용 (0은 거짓, 0이 아니면 참으로 판별)

   - ENUM : 미리 정해진 몇 개의 문자열 값 중 하나만 저장할 수 있는 타입
     + 예를 들어 회원의 등급을 'BRONZE', 'SILVER', 'GOLD' 중 하나로만 제한하고 싶을 때, ENUM('BRONZE', 'SILVER', 'GOLD')와 같이 정의할 수 있지만, 실무에서는 잘 사용하지 않음

   - JSON : JSON 형식의 데이터를 그대로 저장할 수 있는 타입
     + 스키마가 유동적인 데이터를 저장할 때 유용하지만, 관계형 데이터베이스의 장점을 제대로 활용하기 어려워 남용해서는 안 됨
