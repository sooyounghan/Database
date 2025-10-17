-----
### 성능 트레이드 오프
-----
1. 설계 관점에서 자연 키 보다는 대리 키를 사용하는 것이 좋음
2. 자연 키 사용의 장점
   - 장점
     + 조회 단순성 : 비즈니스 요구사항을 보니 자연 키(예) email)만으로 데이터를 조회하는 경우가 대부분이라면, 조인(JOIN) 없이 해당 테이블에서 바로 원하는 정보를 찾을 수 있음
     + 예를 들어 주문 정보에서 회원의 email를 보고 싶을 때, orders 테이블에 email이 직접 저장되어 있다면 member 테이블을 조인할 필요가 없음

3. 예제 : 주문 정보에서 회원 이메일 조회하기
   - 앞서 자연 키의 문제점을 알아볼 때 만들었던 orders_n 테이블과, 대리 키를 사용해 올바르게 설계한 orders_s 테이블을 비교하며 직접 확인
   - 자연 키(member_email)를 사용하는 경우
      + 자연 키를 FK로 사용하는 orders_n 테이블에는 member_email 컬럼이 있어, member_n 테이블과 조인하지 않아도 회원의 이메일을 바로 알 수 있음
```sql
SELECT order_id, member_email, ordered_at
FROM orders_n;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/1e929f30-778d-4725-b155-703fc6955041">
</div>

   - 대리 키(member_id)를 사용하는 경우
     + 반면, 대리 키를 FK로 사용하는 orders_s 테이블에는 member_id만 저장
     + 따라서 회원의 이메일을 확인하려면 반드시 member_s 테이블을 JOIN 해야함
```sql
SELECT
   o.order_id,
   m.email AS member_email,
   o.ordered_at
FROM orders_s o
JOIN member_s m ON o.member_id = m.member_id;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/25f9d41e-7d6c-4e5d-99e4-baa94b605d98">
</div>

   - 참고로 이전 예제에서 회원의 이메일을 ```member1@new.com```으로 변경했기 때문에, 현재 시점의 이메일이 조회되는 것을 확인할 수 있음
   - 자연 키를 사용하는 것이 JOIN을 피할 수 있어 쿼리가 더 단순하고 빨라 보일 수 있음

4. 예제: 특정 이메일로 주문 정보 검색하기
   - 'member1'이라는 이름으로 시작하는 이메일을 사용하는 회원의 모든 주문을 찾는 시나리오를 가정
   - 자연 키(member_email)를 사용하는 경우
      + orders_n 테이블에는 member_email이 있으므로 member_n 테이블을 방문할 필요 없이 orders_n 테이블만으로 검색을 완료할 수 있음
```sql
SELECT *
FROM orders_n
WHERE member_email LIKE 'member1%';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f69117fc-fb02-4db6-b328-dbba9c86392b">
</div>
  
   - 쿼리가 매우 단순하고, orders_n 테이블 하나만 접근하므로 성능 면에서도 유리

   - 대리 키(member_id)를 사용하는 경우
      + orders_s 테이블에는 이메일 정보가 없으므로, 검색 조건인 이메일을 사용하려면 반드시 member_s 테이블과 JOIN 해야함
```sql
SELECT
     o.order_id,
     o.member_id,
     o.ordered_at
FROM orders_s o
JOIN member_s m ON o.member_id = m.member_id
WHERE m.email LIKE 'member1%';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/354d0f92-3155-4ca7-930f-d13645f1dc4a">
</div>

   - 이처럼 자연 키를 검색 조건으로 자주 사용한다면, 해당 자연 키를 FK로 직접 가지고 있는 테이블의 조회 성능이 JOIN을 생략할 수 있어 더 좋을 수 있음

5. 자연 키 사용의 단점
   - 단점 1 - 외래 키 크기 : 이것이 자연 키의 가장 치명적인 단점 중 하나로, 자연 키가 VARCHAR(50) 처럼 긴 문자열이라고 가정
     + 이 키를 참조하는 모든 자식 테이블들(orders, qna, reviews 등 수많은 테이블)의 외래 키 컬럼도 똑같이 VARCHAR(50)이 되어야 함
     + 공간 낭비 : 일반적인 대리 키로 사용하는 BIGINT (8바이트)에 비해 훨씬 많은 디스크 공간을 차지
     + 성능 저하 : 테이블과 인덱스의 크기가 커지면 메모리에 한 번에 올릴 수 있는 데이터 양이 줄어듬
     + 또한 조인 연산 시 비교해야 할 데이터의 크기가 커져 메모리, CPU 사용량도 늘어남
     + 이는 전반적인 데이터베이스 성능 저하로 이어짐

   - 단점2 - 인덱스 단편화와 쓰기 성능 저하 : email, login_id 와 같은 자연 키는 일반적으로 알파벳순이나 가입 순서와 무관하게 생성
     + 예를 들어 apple, mango, banana 순서로 회원이 가입했다고 상상
     + 데이터베이스는 이 값들을 정렬된 상태로 인덱스에 저장해야 하므로, apple 다음에 mango를 넣었다가, 다시 그 사이에 banana를 삽입
     + 이 과정에서 인덱스 페이지를 나누는 페이지 분할(Page Split)이 빈번하게 발생 : 페이지 분할은 새로운 페이지를 할당하고 기존 데이터를 옮기는 복잡하고 비용이 큰 작업
     + 결과적으로 인덱스 내부에 빈 공간이 많이 생기는 단편화(Fragmentation)가 심해지고, 이는 쓰기 성능을 저하

6. 대리 키 사용과 장단점
   - 장점1 - 뛰어난 쓰기 성능 : AUTO_INCREMENT 나 SEQUENCE 를 사용하는 대리 키는 항상 이전 값보다 큰 숫자를 순서대로 만들어냄
     + 새로운 데이터는 인덱스 구조의 가장 마지막에 차례대로 추가(append)되기만 하면 됨
     + 이는 페이지 분할을 거의 일으키지 않으므로 인덱스 단편화 문제가 발생하지 않음
     + 결과적으로 매우 빠르고 효율적인 쓰기 성능을 보장

   - MySQL과 클러스터링 인덱스
     + MySQL의 기본 스토리지 엔진인 InnoDB 스토리지 엔진은 PK에 클러스터링 인덱스라는 특별한 인덱스를 사용
     + 클러스터링 인덱스는 AUTO_INCREMENT처럼 순서대로 생성되는 대리 키 방식에 최적화된 저장 성능을 제공
     
   - 장점2 - 외래 키 크기 최소화 : BIGINT 타입의 대리 키는 8바이트 고정 크기를 가짐
     + 이는 VARCHAR(50) 같은 가변 길이 문자열보다 훨씬 작고 효율적
     + 모든 자식 테이블에서 이 작은 크기의 외래 키를 사용하므로, 테이블과 인덱스의 전체적인 크기를 줄여 디스크 공간을 절약
     + 메모리 효율을 높이고, 조인 시 비교 연산이 빨라져 조회 성능까지 향상시킴

   - 단점1 - 추가 조인 발생 : 앞서 보았듯이 대리 키를 사용하면 특정 비즈니스 데이터를 얻기 위해 추가적인 조인이 필요할수 있음
     + 예를 들어, orders 테이블에는 member_id만 저장되어 있음
     + 만약 주문 목록을 보면서 각 주문을 한 회원의 email을 함께 표시해야 한다면, 반드시 member 테이블을 조인해야만 함
```sql
-- 주문 ID 100번에 대한 회원의 email을 확인하려면 JOIN이 필수
SELECT m.email
 FROM orders o
JOIN member m ON o.member_id = m.member_id
WHERE o.order_id = 100;
```
   - 자연 키를 사용했다면 이 조인은 필요 없었을 것 : 하지만 대부분의 경우, 이 조인으로 인한 약간의 성능 저하보다는 대리 키가 제공하는 다른 수많은 이점(쓰기 성능, 데이터 크기, 유연성)이 훨씬 큼

   - 단점2 - 추가 인덱스 필요 : 대리 키를 기본 키(PK)로 사용한다는 것은, 원래 자연 키로 사용하려던 컬럼(예) email)을 이제 일반 컬럼으로 관리한다는 의미
     + 하지만 사용자는 여전히 email로 로그인을 하고, email로 회원 검색을 해야 함
     + 따라서 이 컬럼의 조회 성능을 보장하고 중복을 방지하기 위해 별도의 UNIQUE 인덱스를 반드시 생성해야 함
```sql
-- member 테이블에는 2개의 인덱스가 필요
-- 1. 기본 키(PK) 인덱스: member_id
-- 2. 고유(UNIQUE) 인덱스: email

CREATE TABLE member (
   member_id BIGINT NOT NULL AUTO_INCREMENT,
   email VARCHAR(50) NOT NULL,
   ...
   PRIMARY KEY (member_id),
   UNIQUE KEY uq_email (email)
);
```
   - 인덱스가 하나 더 늘어난다는 것은 그만큼의 저장 공간을 추가로 사용하고, 데이터를 삽입 / 수정 / 삭제할 때마다 관리해야 할 인덱스가 하나 더 늘어난다는 의미
   - 즉, 약간의 쓰기 성능 저하를 감수해야 함

7. 정리하면 자연 키, 대리 키는 성능 관점에서 각각 장단점이 있지만, 대리 키를 기본 키(PK)로 사용하면 데이터 모델의 안정성, 유연성, 그리고 쓰기 성능이라는 더 큰 이점을 얻을 수 있기 때문에 실무에서는 대부분 대리 키를 기본 키로 선택
