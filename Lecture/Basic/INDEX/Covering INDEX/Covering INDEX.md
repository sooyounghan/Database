-----
### 커버링 인덱스
-----
1. 옵티마이저가 인덱스를 사용하기로 결정해도, 추가적인 작업이 필요함
<div align="center">
<img src="https://github.com/user-attachments/assets/c2d562cb-896e-43be-bb88-48ddeff8162d">
</div>

   - 인덱스 스캔 : idx_items_price 인덱스에서 WHERE 조건에 맞는 데이터(의 위치)를 찾음

2. 테이블 데이터 접근
   - 인덱스에서 찾은 위치 정보(PK인 item_id)를 사용해, 원본 items 테이블에 접근해서 SELECT 절에서 요구하는 다른 컬럼(* 또는 item_name 등)의 데이터를 가져옴
   - 이 과정은 마치 책의 '찾아보기'에서 원하는 키워드와 페이지 번호를 찾은 뒤(1단계), 다시 그 페이지로 직접 넘어가서 내용을 읽는 것(2단계)과 같음
   - 여기서 2단계, 즉 원본 테이블에 다시 접근하는 과정은 랜덤 I/O를 유발하므로 비용이 발생
   - 그렇다면 테이블에 접근하는 이 두 번째 단계를 아예 생략할 방법은 없을까? 쿼리에 필요한 모든 데이터를 인덱스에서만 읽어서 끝낼 수는 없을까? 이 질문에 대한 해답이 바로 커버링 인덱스(Covering Index)

3. 커버링 인덱스
   - 커버링 인덱스는 쿼리에 필요한 모든 컬럼을 포함하고 있는 인덱스
   - '커버링'이라는 이름 그대로, 인덱스 하나가 쿼리의 요구사항 전체를 '덮는다'는 의미
   - 데이터베이스 옵티마이저는 쿼리를 실행할 때, 만약 특정 인덱스가 SELECT, WHERE, ORDER BY, GROUP BY 절에 사용되는 모든 컬럼을 가지고 있다면, 원본 테이블에 전혀 접근하지 않고 오직 인덱스만을 읽어서 쿼리를 처리
   -  이는 디스크의 여러 곳을 오가는 비싼 랜덤 I/O 작업을 완전히 제거하고, 순차 I/O에 가까운 인덱스 스캔만으로 쿼리를 끝낼 수 있음을 의미 (당연히 성능은 비약적으로 향상)

4. 예시 : 커버링 인덱스의 적용 전과 후
   - 쇼핑몰에서 가격이 50,000원에서 100,000원 사이인 상품들의 가격(price)과 이름(item_name)을 조회하는, 매우 흔한 쿼리가 있다고 가정
```sql
SELECT item_id, price, item_name
FROM items
WHERE price BETWEEN 50000 AND 100000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/7a588fc1-c28d-46e0-b3a0-264480ec0703">
</div>

   - 커버링 인덱스 적용 전 (일반 인덱스 사용)
      + 현재 items 테이블에는 price 컬럼에 idx_items_price 인덱스가 걸려 있음
      + 이 상태에서 위 쿼리의 실행 계획을 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/79fc2baf-af4f-4fb3-aa31-acb215443a89">
</div>

```sql
EXPLAIN SELECT item_id, price, item_name
FROM items
WHERE price BETWEEN 50000 AND 100000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8d0f4b53-de0d-474f-af57-c8bb4c64797b">
</div>

   - (key : idx_items_price) : price 조건을 위해 idx_items_price 인덱스를 사용
   - (Extra : Using index condition) : WHERE 조건절을 필터링하는 데 인덱스를 효율적으로 사용했지만, 최종 데이터를 가져오기 위해서는 추가 작업이 필요하다는 의미를 내포
     + 이 쿼리에서는 SELECT 절의 item_name 컬럼이 idx_items_price 인덱스에 포함되어 있지 않음
     + 따라서 옵티마이저는 다음과 같이 동작
        * idx_items_price 인덱스를 스캔하여 price가 조건에 맞는 행의 item_id를 5개 찾음
        * 찾아낸 5개의 item_id를 사용해, items 테이블의 원본 데이터에 5번 접근하여 각각의 item_name을 가져옴 (5번의 랜덤 I/O 발생)

   - 이처럼 인덱스에 포함되지 않은 컬럼(item_name)을 조회해야 하므로, 테이블 접근을 피할 수 없음
   - 참고로 MySQL의 인덱스는 테이블의 기본 키(PK, item_id)를 기본으로 포함
   - 따라서 idx_items_price 인덱스를 사용하는 경우 item_id, price 두 컬럼의 값은 인덱스에서 바로 조회할 수 있음

   - 커버링 인덱스 적용 - 인덱스 컬럼만 조회하는 경우
     + 앞서 idx_items_price 인덱스는 price 컬럼과 기본 키인 item_id를 포함하고 있다고 설명
     + 그렇다면 SELECT 절에서 item_name을 제외하고 item_id와 price만 조회할 경우
       * 이 경우, 쿼리에 필요한 모든 컬럼(item_id, price)이 idx_items_price 인덱스에 이미 포함되어 있으므로, 이 인덱스는 커버링 인덱스의 역할을 수행할 수 있음
       * 실제 쿼리를 실행하면 다음과 같이 item_id와 price만 조회
```sql
SELECT item_id, price
FROM items
WHERE price BETWEEN 50000 AND 100000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/f3de596b-6ab6-4783-b312-79c61c52bb6d">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/7d400b18-42ff-434d-ba37-07fa28f82b2f">
</div>

   - 쿼리에 사용하는 item_id, price 컬럼이 idx_items_price 인덱스에 모두 있음
   - 따라서 items 원본 테이블에 접근할 필요가 없음
     + 결과적으로 items 원본 테이블에 접근하지 않고, idx_items_price 인덱스만 사용
   - item_id 와 price 만 조회하는 쿼리의 실행 계획을 직접 확인
```sql
EXPLAIN SELECT item_id, price
FROM items
WHERE price BETWEEN 50000 AND 100000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/ef286ab0-7a17-426f-8a05-376e8cf3b3a9">
</div>

   - Extra 컬럼에 Using index가 추가된 것을 확인할 수 있음
   - (Extra : Using index) : 이 표시가 가장 중요
     + 이것은 쿼리에 필요한 모든 데이터를 오직 인덱스에서만 읽어서 처리했음을 의미
     + 옵티마이저는 idx_items_price 인덱스만 스캔하여 price와 item_id를 모두 얻었고, 원본 items 테이블에는 전혀 접근할 필요가 없어짐
     + (Extra : Using where) : Using index 와 함께 Using where 가 표시되는 것을 볼 수 있음
       * 이는 인덱스 내에서 WHERE price BETWEEN ... 조건절을 사용해 불필요한 데이터를 필터링했음을 의미
       * Using index와 함께 사용되는 Using where는 테이블에 접근한 후 필터링하는 것이 아니라, 인덱스 스캔 단계에서 효율적으로 필터링이 이루어졌음을 나타냄

   - 결론적으로, 이 실행 계획은 커버링 인덱스를 활용해 테이블 접근을 피했고(Using index), 인덱스 내에서 WHERE 절의 조건으로 필터링(Using where)을 수행한, 매우 효율적인 쿼리임을 보여줌
   - 이처럼 SELECT 절에 어떤 컬럼을 지정하느냐에 따라, 같은 인덱스라도 커버링 인덱스로 동작할 수도 있고 아닐 수도 있음
   - 하지만 우리가 원래 원했던 것은 item_name까지 함께 조회하는 것
   - 이처럼 인덱스에 포함되지 않은 컬럼까지 조회하려면, 쿼리에 필요한 컬럼을 모두 포함하는 새로운 인덱스를 직접 만들어주어야 함

5. 커버링 인덱스 적용 - item_name 추가
   - 이제 커버링 인덱스를 만들어 item_name을 포함하는 쿼리의 성능을 최적화
   - 이 쿼리에 필요한 컬럼은 WHERE 절의 price와 SELECT 절의 item_name : 따라서 이 두 컬럼을 모두 포함하는 인덱스를 생성
   - 💡 컬럼이 여러 개인 복합 인덱스에서 컬럼의 순서는 매우 중요
     + 💡 WHERE 절에서 동등 비교나 범위 검색에 사용되는 컬럼을 가장 앞에 두어야 인덱스를 효율적으로 사용할 수 있음
     + 여기서는 price를 먼저 두고, 그 다음에 item_name 위치
```sql
-- 기존 price 인덱스는 삭제하고, price와 item_name을 포함하는 새로운 복합 인덱스를 생성한다.
DROP INDEX idx_items_price
ON items;

CREATE INDEX idx_items_price_name
ON items (price, item_name);
```
   - 새로운 idx_items_price_name 인덱스는 price로 먼저 정렬되고, price가 같다면 item_name으로 다시 정렬된 구조를 가짐
   - 이제 이 인덱스는 쿼리에 필요한 price와 item_name 정보를 모두 가지고 있음
   - 이 상태에서 다시 한번 동일한 쿼리의 실행 계획을 확인
```sql
EXPLAIN SELECT item_id, price, item_name
FROM items
WHERE price BETWEEN 50000 AND 100000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/56651cc0-3a89-4988-9038-2699692a1855">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/9d8ede24-3595-4410-a89a-f3bd174043b4">
</div>

   - Using index를 통해 인덱스 만으로 쿼리가 실행되는 것을 확인할 수 있음
   - 이 쿼리는 이제 idx_items_price_name 인덱스만 읽고 끝나므로, 테이블 접근으로 인한 랜덤 I/O가 완전히 사라져 훨씬 빠르고 효율적으로 동작
   - 실제 쿼리를 실행해 보면 결과는 이전과 동일
   - 하지만 그 결과를 얻어오는 내부 과정의 효율성은 매우 큰 차이가 발생
```sql
SELECT item_id, price, item_name
FROM items
WHERE price BETWEEN 50000 AND 100000;
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c63babbf-e68a-4a58-84c2-a05923c0fd51">
</div>

6. 커버링 인덱스의 장단점
   - 장점
      + 압도적인 SELECT 성능 향상 : 테이블 접근을 위한 랜덤 I/O를 제거하여 조회 성능을 극적으로 개선
      + 특히 COUNT 쿼리 최적화 : SELECT COUNT(*)와 같은 쿼리에서 테이블 전체가 아닌, 크기가 훨씬 작은 인덱스만 스캔하여 결과를 빠르게 반환할 수 있음
   - 단점
      + 저장 공간 증가: 인덱스는 원본 데이터와 별도의 저장 공간을 차지하며, 인덱스에 포함되는 컬럼이 많아질수록 인덱스의 크기도 커짐
      + 쓰기 성능 저하: INSERT, UPDATE, DELETE 작업 시, 테이블 데이터뿐만 아니라 인덱스도 함께 수정해야 하므로, 인덱스가 많고 복잡할수록 쓰기 작업에 대한 부하가 커짐

7. 언제 사용해야 할까?
   - 커버링 인덱스는 만능 해결책이 아니며, 읽기 성능과 쓰기 성능 사이의 트레이드오프(trade-off)를 신중하게 고려해야 함
   - 조회(읽기)가 매우 빈번하고, 쓰기 작업은 상대적으로 적은 테이블에 적용하는 것이 가장 효과적임
   - SELECT 절에서 조회하는 컬럼의 개수가 적을 때 유리 : SELECT * 처럼 모든 컬럼을 조회하는 쿼리는 커버링 인덱스의 이점을 누리기 어려움 (모든 컬럼을 포함하는 인덱스를 만들 수는 있지만, 이는 사실상 테이블을 복제하는 것과 같아 매우 비효율적)
   - 성능 저하가 발생하는 특정 쿼리를 튜닝하기 위한 '비장의 무기'로 사용하는 경우가 많음
