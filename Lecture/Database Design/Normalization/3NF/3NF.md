-----
### 제 3 정규형
-----
1. 제3 정규형이 필요한 이유
   - orders_2nf 테이블을 분석 : 기본 키는 order_id
<div align="center">
<img src="https://github.com/user-attachments/assets/17ffcb01-bba4-4fed-94ce-06014a0fcdc4">
</div>

2. 함수 종속 관계 분석
   - member_id와 ordered_at는 기본 키인 order_id에 잘 종속
      + order_id → member_id
      + order_id → ordered_at
   - 💡 member_name은 기본 키가 아닌 member_id에 종속
      + member_id → member_name
   - 즉, order_id → member_id → member_name과 같은 종속 관계가 나타남
   - 💡 이처럼 기본 키가 아닌 컬럼이 다른 컬럼을 결정하는 관계를 이행적 함수 종속(Transitive Functional Dependency)
      + 이행적(Transitive) 단어 : '이행적(Transitive)'이라는 단어는 '거쳐서 간다', '옮겨 간다'는 의미

3. 데이터베이스에서는 A → B 이고 B → C 일 때, 결과적으로 A → C가 성립하는 관계를 생각
   - 여기서 A가 C를 결정하는 관계는 B를 거쳐서 가는(이행적인) 관계가 되는 것 : 여기서 A는 일반적으로 테이블의 기본 키
     + 예시에서는 order_id가 member_name을 결정하지만, 중간에 member_id를 거쳐서 감
        * order_id → member_id (주문 ID를 알면 회원 ID를 알 수 있음)
        * member_id → member_name (회원 ID를 알면 회원 이름을 알 수 있음)
     + 따라서 order_id → member_name 관계는 member_id 라는 일반 컬럼을 거쳐가는 이행적 함수 종속 관계
       
   - 제 3 정규형은 바로 이 이행적 함수 종속을 제거하는 과정
   - 이행적 함수 종속의 문제
      + 갱신 이상 : '션' 회원이 '김개발'로 개명하면, member_id가 1인 모든 주문을 찾아 이름을 변경해야 함 - 누락되면 데이터 불일치가 발생
      + 삽입 이상 : 아직 주문을 한 번도 하지 않은 신규 회원은 orders 테이블에 등록할 수 없음 - order_id 가 없기 때문임
      + 삭제 이상 : 1002번 주문을 삭제했는데, 이 주문이 '네이트' 회원의 유일한 주문이었다면 '네이트' 회원 정보 자체가 사라짐

4. 제 3 정규형의 정의와 적용
   - 💡 제 3 정규형 (3NF)
      + 제 2 정규형을 만족해야 함
     + 이행적 함수 종속이 존재하지 않아야 함

  - 이행적 함수 종속을 제거하는 방법 역시 테이블을 분리하는 것 : 결정자(member_id)와 종속자(member_name)를 묶어 별도의 member 테이블로 분리

5. 단계적 설명 및 예제
   - orders_2nf 테이블을 orders_3nf 와 member_3nf 테이블로 분리
```sql
DROP TABLE IF EXISTS orders_3nf;
DROP TABLE IF EXISTS member_3nf;

-- 1. member 테이블 생성 (회원 정보)
CREATE TABLE member_3nf (
 member_id INT NOT NULL,
 member_name VARCHAR(50) NOT NULL,

 PRIMARY KEY (member_id)
);

-- 2. orders 테이블 생성 (주문 정보, member_name 제거)
CREATE TABLE orders_3nf (
 order_id INT NOT NULL,
 member_id INT NOT NULL,
 ordered_at DATETIME NOT NULL,

 PRIMARY KEY (order_id),
 FOREIGN KEY (member_id) REFERENCES member_3nf (member_id)
);

```
   - 분리된 테이블에 데이터를 삽입
```sql
-- member_3nf 데이터 삽입 (회원 정보는 이제 중복 없이 한 번만 저장)
INSERT INTO member_3nf (member_id, member_name) VALUES
(1, '션'),
(2, '네이트');

-- orders_3nf 데이터 삽입
INSERT INTO orders_3nf (order_id, member_id, ordered_at) VALUES
(1001, 1, '2025-08-20 10:00:00'),
(1002, 2, '2025-08-21 11:00:00'),
(1003, 1, '2025-08-21 12:00:00');
```

   - 실행 결과 확인
```sql
SELECT *
FROM member_3nf;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/734b4df9-2f33-4d01-8106-6bf9c5ffcd72">
</div>

   - 실행 결과 확인
```sql
SELECT *
FROM orders_3nf
```
<div align="center">
<img src="https://github.com/user-attachments/assets/70857df1-3fc3-4563-aff5-e19c589bdd50">
</div>

   - 이제 member_name은 member_3nf 테이블에 단 한 번만 저장
   - 만약 '션' 회원의 이름이 바뀌면 member_3nf 테이블의 단 하나의 행만 수정하면 됨
     + 그러면 이 회원을 참조하는 모든 주문에서 변경된 이름 정보를 일관성 있게 조회할 수 있음
   - 데이터 이상 현상이 모두 해결
