-----
### 복합키 설계
-----
1. 경우에 따라서는 하나의 컬럼만으로는 행을 고유하게 식별할 수 없는 상황이 발생 : 이때 등장하는 것이 바로 복합키(Composite Key)
2. 복합키는 왜 필요한가? - 식별자의 조합
   - 복합키가 왜 필요한지 이해하려면, 먼저 데이터 모델링 과정에서 '현실 세계의 데이터를 어떻게 논리적으로 표현할 것인가'라는 근본적인 질문으로 돌아가야 함
   - 모델러들은 각 테이블(엔티티)의 자연스러운 식별자(Natural Key) 즉 자연키를 찾기 위해 노력하는데, 이 식별자가 항상 단일 컬럼으로 표현되지는 않기 때문임
   - 예를 들어 '영화 예매' 시스템을 생각 : '영화'와 '상영 정보', 그리고 '예매'라는 개념을 명확히 구분해야 함
   - 예제를 단순화하기 위해 상영관은 서로 다른 영화를 보여준다고 가정
     + 상영 정보(Screening) : 특정 영화가 특정 날짜, 특정 시간에 상영되는 것을 의미
       * 예를 들어 '매트릭스1'이라는 영화 제목만으로는 특정 상영을 식별할 수 없는데, '매트릭스1'은 여러 날짜와 시간에 걸쳐 수없이 많이 상영되기 때문임
       * 이 상영을 유일하게 특정하려면 '영화 제목'과 '상영 시작 시간'이 함께 필요함
       * 즉, '2025년 8월 31일 19시 30분에 시작하는 매트릭스1'이라고 해야 비로소 하나의 '상영 정보'가 명확히 식별

    + 좌석 예매(Reservation) : 이제 이 '상영 정보'에 좌석을 예매하는 상황을 생각
      * '2025년 8월 31일 19시 30분 매트릭스1'라는 정보만으로는 예매를 특정할 수 없음
      * 왜냐하면 해당 상영 정보에는 수많은 좌석(F8, F9, G8 등)이 존재하고, 각 좌석마다 다른 관객이 예매하기 때문임
      * 따라서 하나의 예매를 고유하게 식별하려면, 어떤 상영의 어떤 좌석인지를 명시해야 함

  - 결론적으로 '좌석 예매'라는 데이터를 식별하기 위한 가장 자연스러운 식별자는 {영화 제목, 상영 시작 시간, 좌석번호} 라는 세 가지 속성의 조합이 됨
  - 이처럼 엔티티의 본질 자체가 복합적인 식별자를 요구하는 상황에 복합키가 필요
  - 복합키는 두 개 이상의 컬럼을 함께 묶어 유일성을 확보하는 방식으로, 전통적 모델링에서 자연 키를 표현하는 핵심적인 방법

3. 복합키를 사용한 테이블 설계
   - 위 개념을 바탕으로, 복합키를 사용한 '영화 예매' 테이블을 설계해 보자. 여기서 '예매' 한 건을 식별하기 위한 기본 키는 {영화 제목, 상영 시작 시간, 좌석 번호} 의 조합이 될 것
   - 테이블 구조 예시 (SQL)
```sql
DROP TABLE IF EXISTS popcorn_order_c;
DROP TABLE IF EXISTS movie_reservation_c;

CREATE TABLE movie_reservation_c (
   movie_title VARCHAR(100) NOT NULL, -- 영화 제목
   screening_dt DATETIME NOT NULL, -- 상영 시작 시간
   seat_number VARCHAR(10) NOT NULL, -- 좌석 번호
   reserver_name VARCHAR(50) NOT NULL, -- 예매자 이름

   -- 복합키: movie_title + screening_dt + seat_number를 기본 키로 설정
   PRIMARY KEY (movie_title, screening_dt, seat_number)
);
```
   - 복합키(Composite Key)를 사용하는 예시이므로 학습 목적으로 테이블을 쉽게 구분하기 위해 뒤에 _c 를 붙임
   - 이 테이블의 기본 키는 movie_title, screening_dt(datetime 줄임), seat_number 세 컬럼의 조합
     + {movie_title, screening_dt} : 이 조합은 하나의 상영 정보를 식별
     + {movie_title, screening_dt, seat_number} : 이 조합은 해당 상영의 특정 좌석에 대한 유일한 예매한 건을 식별
     + 예를 들어, '매트릭스1', '2025-08-31 19:30:00', 'F8'이라는 조합은 단 하나의 예매만을 가리킴
     + 데이터베이스는 이 세 컬럼의 조합이 중복되어 삽입되는 것을 막아 데이터의 유일성을 보장
     + 이 키는 비즈니스 의미(어떤 영화 상영의 어떤 좌석)를 직접 반영하므로 자연 키의 성격을 띰

4. 자연 키를 복합키로 사용하는 예시에서 발견되는 문제점
   - 위 영화 예매 예시에서 복합키를 사용하면 여러 문제점이 드러남
   - 변경 가능성으로 인한 불안정성 : 복합키가 자연 키 기반이므로 비즈니스 변화에 취약함
     + 예를 들어, 영화관 사정으로 상영 시간(screening_dt)이 10분 지연되거나, 영화 제목(movie_title)에 오타가 있어 '매트릭 사 1'에서 '매트릭 스 1'로 수정해야 한다면 기본 키 자체가 바뀌어야 함
     + 이 경우, 이 예매 정보를 참조하는 다른 테이블(예) '예매별 팝콘 주문' 테이블)의 외래 키도 모두 업데이트되어야 하며, 연쇄적인 데이터 수정이 발생
     + 이는 데이터 무결성을 위반할 위험이 크고, 연쇄 변경에 따른 시스템 부하도 증가시킴
   - 외래 키 참조의 복잡성과 크기 증가 : 다른 테이블에서 이 복합키를 외래 키로 참조할 때, 모든 컬럼(movie_title, screening_dt, seat_number)을 복제해야 함
     + 예를 들어, 특정 예매 좌석에 연결된 '팝콘 주문' 테이블에서 외래 키를 설정
```sql
CREATE TABLE popcorn_order_c (
   popcorn_order_id BIGINT NOT NULL AUTO_INCREMENT,
   movie_title VARCHAR(100) NOT NULL, -- FK1
   screening_dt DATETIME NOT NULL, -- FK2
   seat_number VARCHAR(10) NOT NULL, -- FK3
   popcorn_name VARCHAR(50),

   PRIMARY KEY (popcorn_order_id),
   FOREIGN KEY (movie_title, screening_dt, seat_number)
   REFERENCES movie_reservation_c (movie_title, screening_dt, seat_number)
);
```
<div align="center">
<img src="https://github.com/user-attachments/assets/cca4a379-63a1-42c2-9c32-567b75832f96">
</div>

   - 이로 인해 popcorn_order_c 테이블의 크기가 불필요하게 커지고, 조인(JOIN) 쿼리가 복잡해짐
   - 여러 타입(VARCHAR , DATETIME)의 조합은 저장 공간을 더 많이 차지하며, 인덱스 효율성을 떨어뜨려 쓰기 / 조회 성능 저하의 원인이 될 수 있음

   - 조회와 관리의 어려움 : 복합키는 직관적이지 않음
     + 단일 키처럼 간단히 WHERE id = 1 과 같이 조회할 수 없어, 쿼리가 길어지고 오류가 발생하기 쉬움
     + 또한, 실무에서 자주 사용하는 ORM(Object-Relational Mapping) 도구(예) JPA, Hibernate)에서 복합키를 다루려면 별도의 식별자 클래스를 만드는 등 추가적인 매핑 로직이 필요해 개발 복잡도가 증가

   - 확장성 문제 : 시스템이 성장하며 예매를 식별하는 데 '상영관 번호' 같은 새로운 조건이 추가된다면, 기본 키의 구성이 바뀌어야 함
     + 이는 테이블 구조의 대대적인 변경을 의미하며, 관련된 모든 테이블과 쿼리에 영향을 미치는 큰 작업이 됨

5. 조인 쿼리 복잡성 예시
   - 예를 들어, 특정 팝콘 주문(popcorn_order_id 가 101 인)을 한 사람의 이름(reserver_name)을 찾기 위해 popcorn_order_c와 movie_reservation_c 두 테이블을 조인하는 상황을 생각
   - 먼저 샘플 데이터를 입력하고, 복합키를 사용한 조인 쿼리를 실행
```sql
-- 샘플 데이터 삽입
INSERT INTO movie_reservation_c (movie_title, screening_dt, seat_number, reserver_name)
VALUES ('매트릭스1', '2025-08-31 19:30:00', 'F8', '네이트');

INSERT INTO popcorn_order_c (popcorn_order_id, movie_title, screening_dt, seat_number, popcorn_name)
VALUES (101, '매트릭스1', '2025-08-31 19:30:00', 'F8', '카라멜/치토스 팝콘');
```
```sql
SELECT
   mr.reserver_name,
   po.popcorn_order_id,
   po.popcorn_name
FROM
   popcorn_order_c po
JOIN
   movie_reservation_c mr
ON
   po.movie_title = mr.movie_title
   AND po.screening_dt = mr.screening_dt
   AND po.seat_number = mr.seat_number
WHERE
   po.popcorn_order_id = 101;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8926f897-51a2-4ca0-9996-8299903f6e53">
</div>

  - 조인 조건절(ON)에서 movie_title, screening_dt, seat_number 세 개의 컬럼을 모두 비교해야 함
  - 지금은 컬럼이 세 개뿐이지만, 만약 네 개 이상의 컬럼으로 구성된 복합키라면 조인 조건은 훨씬 더 길고 복잡해질 것

6. 대안 - 대리 키
   - 복합키의 문제점을 해결하는 주요 대안은 앞서 살펴본 대리 키(Surrogate Key)를 기본 키로 사용하는 것
   - 대리 키는 비즈니스와 무관한 자동 생성 값(예) AUTO_INCREMENT 를 사용한 BIGINT 타입의 ID)을 활용하며, 불변성을 보장
   - 그리고 대리 키는 간단하게 하나의 컬럼만 사용
   - 영화 예매 예시를 대리 키로 재설계
<div align="center">
<img src="https://github.com/user-attachments/assets/e57ca4a4-79eb-4222-95bb-8e3e22b8c629">
</div>

   - 테이블 구조 예시 (SQL) : 여기서는 대리 키를 사용하는 새로운 테이블( _s 접미사 사용)을 생성
```sql
DROP TABLE IF EXISTS popcorn_order_s;
DROP TABLE IF EXISTS movie_reservation_s;

-- 예매 테이블 (대리 키 사용)
CREATE TABLE movie_reservation_s (
   reservation_id BIGINT NOT NULL AUTO_INCREMENT, -- 대리 키 PK
   movie_title VARCHAR(100) NOT NULL,
   screening_dt DATETIME NOT NULL,
   seat_number VARCHAR(10) NOT NULL,
   reserver_name VARCHAR(50) NOT NULL,

   PRIMARY KEY (reservation_id),
   -- 자연 키 부분에 UNIQUE 제약으로 데이터 무결성 보장
   UNIQUE KEY uq_movie_reservation (movie_title, screening_dt, seat_number)
);
```
```sql
-- 팝콘 주문 테이블 (대리 키 참조)
CREATE TABLE popcorn_order_s (
   popcorn_order_id BIGINT NOT NULL AUTO_INCREMENT,
   reservation_id BIGINT NOT NULL, -- 단순화된 FK
   popcorn_name VARCHAR(50),
   PRIMARY KEY (popcorn_order_id),
   FOREIGN KEY (reservation_id) REFERENCES movie_reservation_s (reservation_id)
);
```
   - 대리 키(Surrogate Key)를 사용하는 예시이므로 학습 목적으로 테이블을 쉽게 구분하기 위해 뒤에 _s를 붙임
   - movie_reservation_s 테이블은 reservation_id 라는 대리 키를 기본 키로 가짐
   - 그리고 비즈니스 유일성을 보장해야 하는 {movie_title, screening_dt, seat_number} 조합에는 UNIQUE 제약조건을 걸어 데이터 무결성을 지킴
   - popcorn_order_s 테이블은 이제 reservation_id 라는 단 하나의 BIGINT 컬럼만 외래 키로 참조하면 됨

<div align="center">
<img src="https://github.com/user-attachments/assets/50e5a91c-71f5-4a9b-b66f-a9589cf7ca0e">
</div>

   - 단순화된 조인 쿼리 예시 : 대리 키를 사용하여 동일한 조회(특정 팝콘 주문을 한 사람의 이름 찾기)를 수행
```sql
-- 샘플 데이터 삽입
INSERT INTO movie_reservation_s (movie_title, screening_dt, seat_number,reserver_name)
VALUES ('매트릭스1', '2025-08-31 19:30:00', 'F8', '네이트');

-- movie_reservation_s 테이블에 방금 입력된 reservation_id는 1
INSERT INTO popcorn_order_s (reservation_id, popcorn_name) VALUES (1, '카라멜/치토스 팝콘');
```
```sql
-- popcorn_order_id가 1인 주문 조회
SELECT
   mr.reserver_name,
   po.popcorn_order_id,
   po.popcorn_name  
FROM
   popcorn_order_s po
JOIN
   movie_reservation_s mr ON po.reservation_id = mr.reservation_id
WHERE
   po.popcorn_order_id = 1;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5153ae70-5830-4880-aa27-e49e2c0f0c17">
</div>

   - 조인 조건이 ON po.reservation_id = mrs.reservation_id 하나로 매우 단순하고 명확해졌음 : 이는 쿼리 가독성을 높이고, 개발자가 실수할 가능성을 줄여줌
   - 이점
     + 불변성 확보 : reservation_id 는 절대 변하지 않으므로, 비즈니스적인 변경(예) 상영 시간 지연, 영화 제목 수정)이 발생해도 키는 안전하며, 다른 테이블에 전파될 영향이 없음
     + 단순한 외래 키 참조: 다른 테이블에서 단일 컬럼(reservation_id)만 참조하면 되므로, 테이블 구조가 단순해지고 저장 공간을 절약할 수 있음
     + 성능 향상: 크기가 작고 타입이 일관된 숫자 키(BIGINT)는 여러 컬럼이 조합된 복합키보다 인덱싱과 조인 연산에서 훨씬 빠름
     + 유연성 및 명확성: 자연 키에 해당하는 {영화 제목, 상영 시간, 좌석 번호} 조합의 유일성은 UNIQUE 제약조건을 통해 비즈니스 규칙으로 계속 강제할 수 있음
       * 물리적인 기본 키의 책임과 비즈니스 식별자의 책임을 분리하여 모델이 더 명확하고 유연해짐

7. 복합키 정리
   - 자연 키를 기본 키로 사용하려면, 비즈니스 로직상 하나의 컬럼만으로는 부족하여 여러 컬럼을 묶은 복합키를 사용해야 하는 경우가 많음
   - 복합키는 데이터를 식별하고 다른 테이블과 관계를 맺을 때 여러 컬럼을 한 묶음으로 다뤄야 하므로 복잡하고 비효율적
   - 반면 대리 키는 비즈니스와 무관한 단 하나의 컬럼으로 데이터를 고유하게 식별
   - 덕분에 다른 테이블과의 관계 역시 이 단일 컬럼 하나로 매우 단순하고 명확하게 표현할 수 있음
   - 이러한 명백한 장점 때문에, 현대적인 데이터베이스 설계에서는 다음의 조합이 사실상의 표준으로 자리 잡음
     + 기본 키 (PK) : 대리 키를 사용 (예) reservation_id)
     + 비즈니스 제약 : 자연 키는 UNIQUE 제약조건으로 설정 (예) UNIQUE(movie_title, screening_dt, seat_number))
   - 이 방식을 통해 데이터 무결성을 확실히 지키면서도, 시스템의 유연성과 확장성을 크게 높일 수 있다. 실무에서는 고민의 여지 없이 대리 키 사용을 우선적으로 고려하는 것이 좋음
