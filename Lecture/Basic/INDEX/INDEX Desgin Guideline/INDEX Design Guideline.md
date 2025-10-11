-----
### 인덱스 설계 가이드라인
-----
1. 인덱스를 만드는 법을 아는 것보다 더 중요한 것은, 어디에 인덱스를 만들어야 하는지 아는 것
   - 잘못된 인덱스는 오히려 시스템 성능을 떨어뜨릴 수 있음
   - 인덱스는 결코 공짜가 아니님 : 데이터를 추가(INSERT), 수정(UPDATE), 삭제(DELETE)할 때마다 인덱스도 함께 변경되어야 하므로 쓰기 성능이 저하되고, 별도의 저장 공간도 차지
   - 따라서 우리는 이 비용을 상쇄하고도 남을 만큼의 '검색 성능 향상'이라는 이득을 얻을 수 있는 곳에만 전략적으로 인덱스를 생성해야 함

2. 핵심 원칙 : 카디널리티 (Cardinality)
   - 인덱스를 어디에 걸지 판단하는 가장 중요한 기준은 바로 카디널리티(Cardinality)
   - 카디널리티(Cardinality) : 해당 컬럼에 저장된 값들의 고유성(uniqueness) 정도를 나타내는 지표
      + 카디널리티가 높음 (High Cardinality) : 해당 컬럼에 중복되는 값이 거의 없다는 의미 (예) items 테이블의 item_id, item_name)
      + 카디널리티가 낮음 (Low Cardinality): 해당 컬럼의 값이 몇 종류 안되어 중복되는 값이 많다는 의미 (예) items 테이블의 category (5종류), is_active (2종류))

   - 인덱스는 '찾아보기'이므로, 찾아보기가 효과적이려면, 특정 키워드를 찾았을 때 검색 범위가 확 줄어들어야 함
     + items 테이블에서 WHERE is_active = TRUE 라는 조건으로 검색한다고 가정
     + is_active 컬럼에 인덱스가 있더라도, TRUE 인 데이터가 전체의 80%라면, 데이터베이스는 인덱스를 통해 전체 데이터의 80%를 스캔해야함 : 이런 경우 데이터베이스 옵티마이저는 풀 테이블 스캔이 적절하다고 판단할 수 있음
     + 반면 WHERE item_name = '게이밍 노트북' : 인덱스는 수십만 건의 상품 데이터 중 단 1건으로 검색 범위를 완벽하게 좁혀줌

   - 💡 핵심 규칙 : 인덱스는 카디널리티가 높은, 즉 식별력이 좋은 컬럼에 생성할 때 가장 효율적

3. 인덱스 생성 가이드라인 : 어떤 컬럼이 인덱스 후보가 되는가?
   - WHERE 절에서 자주 사용되는 컬럼
      + 가장 기본적이고 명백한 가이드라인으로, 인덱스의 존재 이유 자체가 WHERE 절의 검색 속도를 높이는 것이기 때문임
      + 사용자들이 상품을 검색할 때 items.item_name으로 검색하거나, 특정 카테고리(items.category)를 필터링한다면 이 컬럼들은 인덱스 생성의 우선 후보가 됨

   - JOIN의 연결고리가 되는 컬럼 (외래 키)
     + JOIN의 성능은 연결고리가 되는 컬럼에 인덱스가 있는지 여부에 따라 극적으로 달라짐
     + '행복쇼핑' 판매자가 등록한 모든 상품을 조회하는 쿼리 예시
```sql
SELECT
     s.seller_name,
     i.item_name,
     i.price
FROM items i
JOIN sellers s ON i.seller_id = s.seller_id
WHERE s.seller_name = '행복쇼핑';
```

 4. items.seller_id 에 인덱스가 없을 때
    - 만약 items 테이블의 seller_id 컬럼(외래 키)에 인덱스가 없다면, 데이터베이스는 다음과 같이 비효율적으로 동작
    - sellers 테이블에서 seller_name이 '행복쇼핑'인 판매자를 찾음 (seller_id = 1)
    - items 테이블의 모든 행을 처음부터 끝까지 스캔하면서, seller_id가 1인 상품을 하나씩 찾아냄

5. 참고 : 조인의 논리적인 순서와 실제 순서의 차이
    - 💡 SQL의 논리적인 순서는 조인을 모두 다 한 다음에 WHERE를 실행
    - 💡 하지만 데이터베이스는 최적화를위해 먼저 데이터를 줄인 다음에 조인
    - 이때 최종 결과는 논리적인 순서와 같음을 보장
    - items 테이블에 상품이 100만 개 있다면, JOIN을 위해 100만 번의 비교가 일어나는 끔찍한 일이 벌어지며, 풀 테이블 스캔이 발생

6. items.seller_id 에 인덱스가 있을 때
   - items.seller_id에는 외래 키 제약 조건 덕분에 인덱스가 자동으로 생성되어 있음 : 인덱스가 있을 때의 동작은 완전히 다름
     + sellers 테이블에서 seller_name이 '행복쇼핑'인 판매자를 찾음 ( seller_id = 1 )
     + sitems.seller_id 인덱스를 사용하여 seller_id가 1 인 상품 데이터의 위치를 곧바로 찾아냄 (풀 테이블 스캔이 사라지고 몇 번의 탐색만으로 JOIN 이 완료)
```sql
EXPLAIN SELECT
     s.seller_name,
     i.item_name,
     i.price
FROM items i
JOIN sellers s ON i.seller_id = s.seller_id
WHERE s.seller_name = '행복쇼핑';
```
<div align="center">
<img src="https://github.com/user-attachments/assets/54796211-62f9-4ceb-8491-5f8a66384f40">
</div>

   - items 테이블(i)의 type이 ref이고, key가 fk_items_sellers 인 것을 볼 수 있음
   - 이는 JOIN 과정에서 items 테이블을 조회할 때 seller_id 인덱스를 매우 효율적으로 사용했다는 증거
   - 💡 따라서 JOIN에 사용되는 외래 키(Foreign Key) 컬럼에는 반드시 인덱스를 생성해야 함

7. 💡 MySQL은 외래 키 제약조건을 설정하면 인덱스를 자동으로 생성
    - 종종 외래 키 제약조건을 걸지 않고 데이터베이스를 사용하는 경우도 있음
    - 이때는 조인 성능을 위해 외래 키로 사용되는 컬럼에 반드시 인덱스를 직접 생성해야 함

8. ORDER BY 절에서 자주 사용되는 컬럼
   - ORDER BY 를 사용한 정렬은 데이터의 양이 많을 경우 매우 비용이 큰 작업 : 데이터베이스는 결과를 반환하기 전에 모든 데이터를 메모리에 올리고 정렬해야 하기 때문임
   - 만약 ORDER BY 에 사용된 컬럼에 인덱스가 있다면 어떨까?
     + B-Tree 인덱스는 이미 데이터가 정렬된 상태로 저장되어있음
     + 데이터베이스는 굳이 데이터를 따로 정렬할 필요 없이, 인덱스에 있는 순서 그대로 데이터를 읽기만 하면 됨
     + 비용이 큰 정렬 작업(filesort)을 완전히 건너뛸 수 있는 것

   - '최신 등록 상품 목록 10개'를 보여주는 ORDER BY registered_date DESC LIMIT 10과 같은 쿼리는 registered_date 컬럼에 인덱스가 있을 때 엄청난 성능 향상을 기대할 수 있음
